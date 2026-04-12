# Week 18, Day 3: Refunds

By the end of today, the admin can refund a paid order and the money actually flows back to the customer's phone or card. The refund path is different on each provider and you will implement all three. You will also track refund state in the database so a half-refunded order does not get refunded twice.

**Prior-week concepts you will use today:**
- The unified payments interface (Week 17, Day 3)
- Idempotency and atomic updates (Week 18, Day 2)
- Daraja B2C (new today)
- Stripe refund API (new today)

**Estimated time:** 3 hours

---

## Why Refunds Are Hard

A charge is "push payment from customer to merchant". A refund is "push payment from merchant back to customer". The directions are different, the APIs are different, and the failure modes are different.

**On M-Pesa**, charges use STK Push / C2B and refunds use a completely different API called B2C (Business to Customer). B2C requires a different API credential, a different initiator name, a different shortcode. It is not "just reverse the transaction" -- it is a fresh outbound transaction.

**On Airtel**, refunds are called "reversals" and have yet another set of endpoints.

**On Stripe**, refunds are a single API call against the original charge -- the cleanest of the three.

Each has its own failure modes too. A refund can fail because:
- The original charge was not actually paid (you never had the money).
- The customer's account was closed between charge and refund.
- The merchant does not have sufficient float (common on B2C -- you need money on your paybill for B2C to pay out).
- The provider is down.

Every refund must be treated as a new transaction with its own state, logs, and retries.

---

## Refund Schema

```sql
CREATE TABLE refunds (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id),
  amount_cents INTEGER NOT NULL CHECK (amount_cents > 0),
  reason TEXT,
  method TEXT NOT NULL CHECK (method IN ('mpesa', 'airtel', 'stripe')),
  provider_reference TEXT,
  provider_transaction_id TEXT,
  status TEXT NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'initiated', 'success', 'failed')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  created_by UUID REFERENCES users(id)
);
CREATE INDEX idx_refunds_order_id ON refunds(order_id);
CREATE INDEX idx_refunds_status ON refunds(status);
```

Each refund row references the order and tracks its own state machine (`pending -> initiated -> success/failed`). An order can have multiple refund rows if you do partial refunds.

Add a helper:

```javascript
async function getTotalRefunded(orderId) {
  const { rows } = await query(
    `SELECT COALESCE(SUM(amount_cents), 0)::int AS total
     FROM refunds
     WHERE order_id = $1 AND status = 'success'`,
    [orderId]
  );
  return rows[0].total;
}
```

Rule: `total_refunded <= subtotal_cents`. Enforced in code before initiating any new refund.

---

## Stripe Refunds (Start Here Because They Are Simple)

```javascript
// server/services/payments/stripeProvider.js
async function refund({ orderId, amountCents, reason }) {
  const { rows } = await query(
    "SELECT stripe_session_id FROM orders WHERE id = $1",
    [orderId]
  );
  const sessionId = rows[0]?.stripe_session_id;
  if (!sessionId) throw new Error("No Stripe session for this order");

  // Get the payment_intent id from the session
  const session = await stripe.checkout.sessions.retrieve(sessionId);
  const paymentIntentId = session.payment_intent;

  const refund = await stripe.refunds.create({
    payment_intent: paymentIntentId,
    amount: amountCents,
    reason: reason || "requested_by_customer",
  });

  return {
    providerReference: refund.id,
    status: refund.status === "succeeded" ? "success" : "pending",
  };
}
```

Stripe refund states:
- `pending` -- accepted, not yet processed
- `succeeded` -- done
- `failed` -- denied or card issue
- `canceled` -- cancelled before completion

Stripe sends a `charge.refunded` webhook when the refund finalises. Add a case to your existing webhook handler:

```javascript
case "charge.refunded": {
  const charge = event.data.object;
  const refund = charge.refunds.data[charge.refunds.data.length - 1];
  await query(
    "UPDATE refunds SET status = 'success', updated_at = NOW() WHERE provider_reference = $1",
    [refund.id]
  );
  break;
}
```

---

## M-Pesa B2C Refunds

B2C requires a whole new set of credentials and a registered shortcode that can *send* money (not just receive). On sandbox, Safaricom provides test B2C credentials.

```env
MPESA_B2C_INITIATOR_NAME=testapi
MPESA_B2C_INITIATOR_PASSWORD=Safaricom999!*!
MPESA_B2C_SHORTCODE=600999
MPESA_B2C_SECURITY_CREDENTIAL=...  # generated from the password
```

The `SecurityCredential` is a public-key-encrypted version of the initiator password. Sandbox gives you a pre-generated one; production requires you to encrypt with Safaricom's certificate. The encryption script:

```javascript
const crypto = require("crypto");
const fs = require("fs");

function generateSecurityCredential(password, certPath) {
  const cert = fs.readFileSync(certPath);
  const encrypted = crypto.publicEncrypt(
    {
      key: cert,
      padding: crypto.constants.RSA_PKCS1_PADDING,
    },
    Buffer.from(password)
  );
  return encrypted.toString("base64");
}
```

Run once at boot and cache the result in an env var. The certificate is downloaded from Safaricom's developer portal.

### The refund call

```javascript
// server/services/payments/mpesaProvider.js
async function refund({ orderId, amountCents, reason }) {
  const token = await getDarajaToken();
  const { rows } = await query(
    "SELECT customer_phone FROM orders WHERE id = $1",
    [orderId]
  );
  const phone = rows[0]?.customer_phone;
  if (!phone) throw new Error("Order not found");

  const payload = {
    InitiatorName: env.MPESA_B2C_INITIATOR_NAME,
    SecurityCredential: env.MPESA_B2C_SECURITY_CREDENTIAL,
    CommandID: "BusinessPayment",
    Amount: Math.ceil(amountCents / 100),
    PartyA: env.MPESA_B2C_SHORTCODE,
    PartyB: formatPhone(phone),
    Remarks: reason || "Refund",
    QueueTimeOutURL: `${env.PUBLIC_URL}/api/payments/b2c/timeout`,
    ResultURL: `${env.PUBLIC_URL}/api/payments/b2c/result`,
    Occasion: `refund-${orderId.slice(0, 8)}`,
  };

  const res = await axios.post(
    "https://sandbox.safaricom.co.ke/mpesa/b2c/v3/paymentrequest",
    payload,
    { headers: { Authorization: `Bearer ${token}` } }
  );

  if (res.data.ResponseCode !== "0") {
    throw new Error(`B2C rejected: ${res.data.ResponseDescription}`);
  }

  return {
    providerReference: res.data.ConversationID,
    status: "initiated",
  };
}
```

Two callbacks: a `ResultURL` for the actual result and a `QueueTimeOutURL` if the request times out in Safaricom's queue. Both are public endpoints you implement.

Route them in the Express server:

```javascript
router.post("/b2c/result", express.json(), asyncHandler(async (req, res) => {
  res.json({ ResultCode: 0, ResultDesc: "Accepted" });

  const result = req.body?.Result;
  if (!result) return;

  const conversationId = result.ConversationID;
  const resultCode = result.ResultCode;

  if (resultCode === 0) {
    // Success
    const metadata = {};
    for (const item of result.ResultParameters?.ResultParameter || []) {
      metadata[item.Key] = item.Value;
    }
    await query(
      `UPDATE refunds
       SET status = 'success',
           provider_transaction_id = $1,
           updated_at = NOW()
       WHERE provider_reference = $2`,
      [metadata.TransactionReceipt, conversationId]
    );
  } else {
    await query(
      `UPDATE refunds SET status = 'failed', updated_at = NOW() WHERE provider_reference = $1`,
      [conversationId]
    );
  }
}));
```

---

## Airtel Reversals

Airtel's reversal endpoint is straightforward but requires the original transaction id:

```javascript
async function refund({ orderId, amountCents }) {
  const token = await getAirtelToken();
  const { rows } = await query(
    "SELECT airtel_transaction_id FROM orders WHERE id = $1",
    [orderId]
  );
  const txnId = rows[0]?.airtel_transaction_id;
  if (!txnId) throw new Error("No Airtel transaction id");

  const res = await axios.post(
    `${BASE_URL}/standard/v2/payments/refund`,
    { transaction: { airtel_money_id: txnId } },
    { headers: { Authorization: `Bearer ${token}`, "X-Country": "KE", "X-Currency": "KES" } }
  );

  if (res.data.status?.code !== "200") {
    throw new Error(`Airtel reversal failed: ${res.data.status?.message}`);
  }

  return {
    providerReference: res.data.data?.transaction?.airtel_money_id,
    status: "success",
  };
}
```

Airtel reversals are typically synchronous in their API -- the response tells you whether it worked. No async callback. This is easier than M-Pesa but does not follow the same pattern, so your Refund service has to handle both "sync success" and "async pending" response shapes.

---

## The Refund Service

Put them all behind one function:

```javascript
// server/services/refunds.service.js
const payments = require("./payments");
const { query, pool } = require("../config/db");

async function createRefund({ orderId, amountCents, reason, createdBy }) {
  // 1. Load the order and check refundability
  const { rows } = await query(
    "SELECT id, subtotal_cents, payment_method, status, payment_status FROM orders WHERE id = $1",
    [orderId]
  );
  const order = rows[0];
  if (!order) throw new Error("Order not found");
  if (order.payment_status !== "success") throw new Error("Order was not paid");
  if (order.status === "cancelled") throw new Error("Order already cancelled");

  // 2. Check total refunded so far
  const already = await getTotalRefunded(orderId);
  if (already + amountCents > order.subtotal_cents) {
    throw new Error("Refund amount exceeds remaining");
  }

  // 3. Create the refund row in pending
  const client = await pool.connect();
  let refundId;
  try {
    await client.query("BEGIN");
    const { rows: refundRows } = await client.query(
      `INSERT INTO refunds (order_id, amount_cents, reason, method, status, created_by)
       VALUES ($1, $2, $3, $4, 'pending', $5)
       RETURNING id`,
      [orderId, amountCents, reason, order.payment_method, createdBy]
    );
    refundId = refundRows[0].id;
    await client.query("COMMIT");
  } finally {
    client.release();
  }

  // 4. Call the provider
  try {
    const provider = payments.getProvider(order.payment_method);
    const result = await provider.refund({ orderId, amountCents, reason });

    await query(
      `UPDATE refunds
       SET status = $1, provider_reference = $2, updated_at = NOW()
       WHERE id = $3`,
      [result.status === "success" ? "success" : "initiated", result.providerReference, refundId]
    );

    return { refundId, status: result.status };
  } catch (err) {
    await query(
      `UPDATE refunds SET status = 'failed', updated_at = NOW() WHERE id = $1`,
      [refundId]
    );
    throw err;
  }
}
```

Workflow:
1. Validate the order can be refunded.
2. Check we are not over-refunding.
3. Insert a `pending` refund row (so we have a database record before the API call).
4. Call the provider.
5. Update the refund row based on the result.

Steps 3-5 are why refunds need their own row rather than a column on the order: you can have multiple partial refunds, you have a clean history, and failures do not stop future attempts.

---

## Admin Refund UI

Add a "Refund" button to the admin order detail. The Server Action:

```javascript
// app/actions/refunds.js
"use server";
import { requireAdmin } from "./auth";

export async function refundOrder(orderId, amountKshInput, reason) {
  const user = await requireAdmin();
  const amountCents = Math.round(parseFloat(amountKshInput) * 100);
  if (!amountCents || amountCents <= 0) return { error: "Invalid amount" };

  try {
    const res = await fetch(`${process.env.CRM_SERVER_URL}/api/refunds`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ orderId, amountCents, reason, createdBy: user.sub }),
    });
    const data = await res.json();
    if (!res.ok) return { error: data.error || "Refund failed" };
    return { refundId: data.refundId, status: data.status };
  } catch (err) {
    return { error: "Service unavailable" };
  }
}
```

And a form on the order page:

```jsx
<form action={async (fd) => {
  const amount = fd.get("amount");
  const reason = fd.get("reason");
  const result = await refundOrder(order.id, amount, reason);
  if (result.error) alert(result.error);
}}>
  <input name="amount" type="number" step="0.01" placeholder="Amount (KSh)" required />
  <input name="reason" type="text" placeholder="Reason" required />
  <button type="submit" className="bg-red-600 text-white px-4 py-2">Refund</button>
</form>
```

Show a list of existing refunds below so the admin knows what is already refunded.

---

## Checkpoint

1. Refunding a paid M-Pesa order (sandbox B2C) triggers the result callback and the refund row flips to `success`.
2. Refunding a Stripe order hits the refunds API and the webhook updates the row.
3. Attempting to refund more than the order subtotal returns an error.
4. Partial refunds stack: refund KSh 500 twice on a KSh 2000 order, total_refunded is KSh 1000, and a third refund of KSh 1500 is rejected.
5. Refunding the same order twice with the same amount creates two distinct refund rows (if both are valid) -- the uniqueness is per refund, not per order.
6. Failed refunds (e.g., provider returns an error) leave the refund row in `failed` with no partial effect.
7. Admin UI shows a clean list of refunds on each order detail.

Commit:

```bash
git add .
git commit -m "feat: refunds across mpesa airtel and stripe with admin ui"
```

---

## What You Learned

- Refunds are new transactions in the opposite direction, with their own state machine.
- Each provider's refund API is different in shape and timing.
- Partial refunds must be bounded by total refundable amount.
- Async refund results go through callbacks just like charges.
- One `refunds` table captures the history across providers.

Tomorrow we build reconciliation -- comparing what the providers say happened with what our database says happened, daily, and catching the gaps.
