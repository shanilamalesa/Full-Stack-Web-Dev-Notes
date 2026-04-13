# Week 16, Day 1: M-Pesa Inside the Shop

> **AI boundaries this week:** 30% manual / 70% AI. Habit: *AI as integration engineer -- you direct the wiring, AI runs it, you verify every connection.* The money rule from Week 10 still holds. See [ai.md](../ai.md).

By the end of today, clicking "Pay" on your Next.js shop triggers a real M-Pesa STK Push to the customer's phone. The order is marked `pending` in the database, the customer gets a push notification, they enter their PIN, and the payment request goes through Daraja. You will not yet handle the callback that marks the order `paid` -- that is tomorrow. Today is wiring the outgoing leg.

You will reuse most of the M-Pesa code from Week 10. The new work is integrating it cleanly into the Next.js Server Action flow instead of the standalone Express endpoint.

**Prior-week concepts you will use today:**
- M-Pesa Daraja basics: consumer key/secret, OAuth token, STK Push (Week 10)
- Server Actions and transactions (Week 15, Day 3)
- The shop's `orders` schema (Week 15, Day 3)

**Estimated time:** 3-4 hours

---

## The Week Ahead

| Day | Focus |
|---|---|
| Day 1 (today) | STK Push from the Next.js shop. Outgoing leg only. |
| Day 2 | M-Pesa callback webhook, order status updates, customer polling. |
| Day 3 | WhatsApp order confirmations via the Week 11 bot -- reuse. |
| Day 4 | Admin notifications, failed-payment retries, refund basics. |
| Day 5 | Recap. |
| Weekend | **Project 3**: E-commerce Lite + WhatsApp Updates, full-stack. |

---

## Where M-Pesa Actually Lives

Recall from Week 10: M-Pesa Daraja is an HTTP API. You get an OAuth token, you call STK Push with a phone number and amount, Safaricom pushes a PIN prompt to the customer's phone, and when the customer enters the PIN Safaricom calls a webhook on your server with the result.

Two servers are involved in the shop now:

1. **The Next.js shop** (`shop/`) -- runs the checkout and the Server Action.
2. **The Express CRM server** (`server/`) -- holds the Week 10 Daraja code, the WhatsApp webhook, the USSD endpoint. Still running.

Where should the M-Pesa integration live? Three options:

- **In the Next.js Server Action.** Clean, one codebase. Downside: the Daraja callback URL is public, and Next.js Server Actions are not designed to handle third-party webhooks -- they are for client-initiated calls.
- **In the Express server.** Reuses Week 10 code exactly. The Next.js action calls an HTTP endpoint on the Express server to kick off STK Push; Daraja calls back to Express; Express updates Postgres.
- **Split:** Next.js calls Daraja directly for the STK Push, Express handles the callback webhook.

We go with **option 2**. Why: the Express server already has a working Daraja integration, a publicly reachable webhook URL, signature verification (or will have by Week 18), and logging. Running STK Push from Next.js would duplicate the Daraja token management. Having the Express server do all the payment work is exactly the "shared backend service" pattern real companies use.

So the flow becomes:

```
customer clicks Pay
  -> Next.js Server Action createOrder()
       writes order row (status: pending_payment)
       POSTs to Express http://localhost:5000/api/payments/stk
  -> Express receives the request
       fetches Daraja token
       calls Daraja STK Push
       returns { checkoutRequestId } to Next.js
  -> Server Action returns { orderId, checkoutRequestId } to client
  -> Client navigates to /checkout/awaiting-payment
  -> Customer enters PIN on their phone
  -> Daraja calls Express /api/payments/callback
  -> Express updates order row -> status: paid
  -> Client page polls /api/orders/:id every 2s, sees status: paid
  -> Navigate to /checkout/confirmed
```

That is the complete dance. Five systems on the wire.

---

## A New Table: `payments`

Before code, one schema addition. Orders get a payment column:

```sql
ALTER TABLE orders
  ADD COLUMN mpesa_checkout_request_id TEXT,
  ADD COLUMN mpesa_receipt_number TEXT,
  ADD COLUMN payment_status TEXT NOT NULL DEFAULT 'not_initiated'
    CHECK (payment_status IN ('not_initiated', 'initiated', 'success', 'failed', 'cancelled'));

CREATE INDEX idx_orders_checkout_request ON orders(mpesa_checkout_request_id);
```

Two decisions worth naming.

**`mpesa_checkout_request_id` is indexed and will be the key** the callback uses to find the right order. When Daraja calls back, you get a `CheckoutRequestID` and have to find the order you created with it. Index it.

**`payment_status` is separate from `status`.** The main `status` is the order lifecycle (pending -> paid -> shipped -> delivered). `payment_status` is the payment lifecycle. An order can be in status `pending` with payment `initiated`, or `paid` with payment `success`. Keeping them separate means you can retry payments without changing the order status, and you can reconcile easily.

---

## Express Side: A Payments Endpoint

Add a route in the Week 10 `server/` project that handles STK Push initiation for the shop. Create `server/routes/payments.routes.js`:

```javascript
// server/routes/payments.routes.js
const express = require("express");
const router = express.Router();
const paymentsController = require("../controllers/payments.controller");
const asyncHandler = require("../middleware/asyncHandler");

router.post("/stk", asyncHandler(paymentsController.initiateStkPush));
router.post("/callback", express.json(), asyncHandler(paymentsController.handleCallback));
router.get("/status/:checkoutRequestId", asyncHandler(paymentsController.getStatus));

module.exports = router;
```

Create `server/controllers/payments.controller.js`:

```javascript
const paymentsService = require("../services/payments.service");

async function initiateStkPush(req, res) {
  const { orderId, phone, amount } = req.body;
  const result = await paymentsService.initiateStkPush({ orderId, phone, amount });
  res.json(result);
}

async function handleCallback(req, res) {
  await paymentsService.handleCallback(req.body);
  res.json({ ResultCode: 0, ResultDesc: "Accepted" });
}

async function getStatus(req, res) {
  const status = await paymentsService.getStatus(req.params.checkoutRequestId);
  res.json(status);
}

module.exports = { initiateStkPush, handleCallback, getStatus };
```

Create `server/services/payments.service.js` (mostly a refactor of Week 10 code):

```javascript
const axios = require("axios");
const { query } = require("../config/db");
const env = require("../config/env");

async function getDarajaToken() {
  const auth = Buffer.from(`${env.MPESA_CONSUMER_KEY}:${env.MPESA_CONSUMER_SECRET}`).toString("base64");
  const res = await axios.get(
    "https://sandbox.safaricom.co.ke/oauth/v1/generate?grant_type=client_credentials",
    { headers: { Authorization: `Basic ${auth}` } }
  );
  return res.data.access_token;
}

function formatPhone(phone) {
  // Daraja wants 2547xxxxxxxx (no +)
  return phone.replace(/^\+/, "").replace(/^0/, "254");
}

async function initiateStkPush({ orderId, phone, amount }) {
  const token = await getDarajaToken();
  const timestamp = new Date().toISOString().replace(/[-T:.Z]/g, "").slice(0, 14);
  const password = Buffer.from(
    env.MPESA_SHORTCODE + env.MPESA_PASSKEY + timestamp
  ).toString("base64");

  const payload = {
    BusinessShortCode: env.MPESA_SHORTCODE,
    Password: password,
    Timestamp: timestamp,
    TransactionType: "CustomerPayBillOnline",
    Amount: Math.ceil(amount / 100), // amount is in cents, Daraja wants KSh
    PartyA: formatPhone(phone),
    PartyB: env.MPESA_SHORTCODE,
    PhoneNumber: formatPhone(phone),
    CallBackURL: `${env.PUBLIC_URL}/api/payments/callback`,
    AccountReference: orderId.slice(0, 12),
    TransactionDesc: `Order ${orderId.slice(0, 8)}`,
  };

  const res = await axios.post(
    "https://sandbox.safaricom.co.ke/mpesa/stkpush/v1/processrequest",
    payload,
    { headers: { Authorization: `Bearer ${token}` } }
  );

  const { CheckoutRequestID, ResponseCode } = res.data;

  if (ResponseCode !== "0") {
    throw new Error(`STK push rejected: ${res.data.ResponseDescription}`);
  }

  await query(
    `UPDATE orders
     SET mpesa_checkout_request_id = $1, payment_status = 'initiated', updated_at = NOW()
     WHERE id = $2`,
    [CheckoutRequestID, orderId]
  );

  return { checkoutRequestId: CheckoutRequestID };
}

async function getStatus(checkoutRequestId) {
  const { rows } = await query(
    `SELECT id, payment_status, status FROM orders WHERE mpesa_checkout_request_id = $1`,
    [checkoutRequestId]
  );
  return rows[0] || null;
}

async function handleCallback(body) {
  // Stub for today; real implementation is Day 2
  console.log("M-Pesa callback received:", JSON.stringify(body));
}

module.exports = { initiateStkPush, handleCallback, getStatus };
```

Wire the route in `server/index.js`:

```javascript
app.use("/api/payments", require("./routes/payments.routes"));
```

Restart the Express server. You now have three endpoints:

- `POST /api/payments/stk` -- the Next.js action will call this.
- `POST /api/payments/callback` -- Daraja calls this. (Stub today.)
- `GET /api/payments/status/:id` -- the Next.js client will poll this.

### Public URL

The Daraja callback needs a public URL. ngrok the Express server (not the Next.js server) on a different port than your WhatsApp ngrok:

```bash
ngrok http 5000
```

Copy the HTTPS URL into `.env` as `PUBLIC_URL=https://xyz.ngrok-free.app`. Restart Express.

---

## Next.js Side: Updating `createOrder`

Back in the shop project, modify `app/actions/createOrder.js` to call the Express payments endpoint after inserting the order:

```javascript
// inside createOrder, after COMMIT:
if (paymentMethod === "mpesa") {
  try {
    const stkRes = await fetch(`${process.env.CRM_SERVER_URL}/api/payments/stk`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        orderId,
        phone: customer.phone,
        amount: subtotalCents,
      }),
    });
    const stkData = await stkRes.json();
    if (!stkRes.ok) {
      console.error("STK push failed:", stkData);
      return { orderId, error: "Payment initiation failed. You can retry from the order page." };
    }
    return { orderId, checkoutRequestId: stkData.checkoutRequestId };
  } catch (err) {
    console.error("STK push error:", err);
    return { orderId, error: "Could not reach payment service." };
  }
}

return { orderId }; // for COD orders
```

Add `CRM_SERVER_URL=http://localhost:5000` to `.env.local`.

Notice we return `{ orderId, error }` together if payment init failed. The order exists (we committed), but the payment didn't start. The client can navigate to a retry page. We do not roll back the order -- the row is a record of intent, and rolling it back would lose the stock deduction. Instead, the order stays in `payment_status = 'initiated'` (well, actually it never got there) and the admin can retry later.

---

## The Awaiting-Payment Page

Create `app/checkout/awaiting-payment/page.js` and `AwaitingPaymentClient.js`:

```jsx
// app/checkout/awaiting-payment/page.js
import AwaitingPaymentClient from "./AwaitingPaymentClient";
export const metadata = { title: "Awaiting Payment" };
export default function Page() {
  return (
    <div className="max-w-md mx-auto p-16 text-center">
      <AwaitingPaymentClient />
    </div>
  );
}
```

```jsx
// app/checkout/awaiting-payment/AwaitingPaymentClient.js
"use client";
import { useEffect, useState } from "react";
import { useRouter } from "next/navigation";
import { useCheckout } from "@/app/lib/checkoutStore";

export default function AwaitingPaymentClient() {
  const router = useRouter();
  const orderId = useCheckout((s) => s.orderId);
  const checkoutRequestId = useCheckout((s) => s.checkoutRequestId);
  const [message, setMessage] = useState("Check your phone for an M-Pesa prompt.");
  const [polling, setPolling] = useState(true);

  useEffect(() => {
    if (!orderId || !checkoutRequestId) {
      router.replace("/");
      return;
    }

    const interval = setInterval(async () => {
      const res = await fetch(
        `${process.env.NEXT_PUBLIC_CRM_URL}/api/payments/status/${checkoutRequestId}`
      );
      const data = await res.json();
      if (data?.payment_status === "success") {
        clearInterval(interval);
        setPolling(false);
        router.push("/checkout/confirmed");
      } else if (data?.payment_status === "failed" || data?.payment_status === "cancelled") {
        clearInterval(interval);
        setPolling(false);
        setMessage("Payment failed or was cancelled. Please try again.");
      }
    }, 2000);

    // Stop polling after 2 minutes
    const timeout = setTimeout(() => {
      clearInterval(interval);
      setPolling(false);
      setMessage("Payment timed out. If you completed the payment, it may still arrive.");
    }, 120000);

    return () => {
      clearInterval(interval);
      clearTimeout(timeout);
    };
  }, [orderId, checkoutRequestId, router]);

  return (
    <>
      <div className="text-4xl mb-4">{polling ? "⏳" : ""}</div>
      <h1 className="text-2xl font-bold">Awaiting Payment</h1>
      <p className="mt-4 text-gray-700">{message}</p>
      {!polling && (
        <button
          onClick={() => router.push(`/my-orders?phone=`)}
          className="mt-6 bg-brand text-white px-6 py-3 rounded"
        >
          Check my orders
        </button>
      )}
    </>
  );
}
```

Add `checkoutRequestId` to the checkout store (alongside `orderId`).

Update the payment form `handlePay` to navigate to `/checkout/awaiting-payment` on success instead of `/checkout/confirmed` (unless it is COD).

### On polling

Polling every 2 seconds is fine for a CRM with dozens of concurrent checkouts. For hundreds, switch to Server-Sent Events (SSE) or WebSockets. We will not do that in this Marathon; polling at this scale is acceptable and much simpler.

---

## Checkpoint

1. Start all three servers: Postgres, Express (with the payments routes), Next.js shop.
2. Add ngrok for Express; put the URL in `.env` as `PUBLIC_URL`.
3. Buy a test product on the Next.js shop with a real Kenyan test number (sandbox accepts `254708374149`).
4. Within a few seconds, the sandbox simulator (or your real phone if you are on a live Daraja project) receives an STK prompt.
5. The orders table shows the row with `payment_status = 'initiated'` and a non-null `mpesa_checkout_request_id`.
6. The "Awaiting Payment" page is polling Express `/api/payments/status/...` every 2 seconds (watch the Express log).
7. Since the callback is still stubbed, the status never updates. That is expected -- tomorrow you wire the callback.

Commit:

```bash
git add .
git commit -m "feat: mpesa stk push from nextjs shop through express payments service"
```

---

## What You Learned

- The shop initiates payments through a shared backend service, not from its own process.
- Order rows exist before payment succeeds; `payment_status` tracks the payment leg.
- `CheckoutRequestID` is the key linking an order to its Daraja session.
- Polling from the client is acceptable for low-volume apps and much simpler than sockets.
- Environment separation: Next.js `.env.local` has the CRM URL; Express `.env` has the Daraja secrets.

Tomorrow we handle the callback webhook, update order status, and close the loop so the customer actually sees the confirmation page.
