# Week 18, Day 2: Idempotency Deep Dive

By the end of today, every mutation in your payment layer is idempotent. Retries by the provider, retries by the customer, duplicate button clicks, late callbacks, out-of-order callbacks, process crashes mid-write -- none of them can corrupt the order state. You will also learn the standard "idempotency key" pattern that big payment systems use.

**Prior-week concepts you will use today:**
- Webhook handlers for M-Pesa, Airtel, Stripe (all weeks)
- Transactions with `pool.connect()` + `BEGIN/COMMIT` (Week 15, Day 3)
- Unique constraints and `ON CONFLICT` (Week 12)

**Estimated time:** 3 hours

---

## What "Idempotent" Actually Means

An operation is **idempotent** if running it N times has the same effect as running it once. `SET status = 'paid'` is idempotent. `UPDATE stock = stock - 1` is not -- running it twice subtracts twice.

In payments you want every operation you expose to the public network to be idempotent. Providers retry. Networks drop. Users double-click. The only way to stay correct is to make your handlers ignore duplicates cleanly.

There are two common strategies:

1. **Natural idempotency**: check state before acting.  "Already paid? Do nothing."
2. **Idempotency keys**: require every request to include a unique key; reject duplicates.

You have been doing (1) all along. Today we add (2) for the cases that need it.

---

## Natural Idempotency: The Pattern You Already Use

Every webhook handler does this:

```javascript
if (order.payment_status === "success") {
  return; // Already processed, skip.
}
```

That works for callbacks because the provider retries the *exact same event*, and the state change is the same every time. But it has blind spots:

**Blind spot 1: the check and the write are not atomic.** Between `SELECT` and `UPDATE`, another callback can land. Two concurrent callbacks see `payment_status = 'initiated'`, both pass the check, both run the update. You do not double-charge (the database is still consistent) but you might double-fire the WhatsApp notification.

**Blind spot 2: partial failures mid-transaction.** You mark the order `paid`, then crash before firing the WhatsApp. The retry finds `paid` and skips the notification entirely. The customer never gets their confirmation.

**Blind spot 3: out-of-order callbacks.** The `shipped` callback arrives before the `paid` callback because of network quirks. Your handler sees status `pending`, the `shipped` transition is invalid, and the callback errors out.

Each blind spot has a fix. Let us do them in turn.

---

## Fix 1: Atomic Check-And-Set

Collapse the check and update into a single SQL statement using a `WHERE` clause:

```javascript
async function markPaid(orderId, receiptNumber) {
  const result = await query(
    `UPDATE orders
     SET payment_status = 'success',
         status = 'paid',
         mpesa_receipt_number = $1,
         updated_at = NOW()
     WHERE id = $2
       AND payment_status != 'success'
     RETURNING id`,
    [receiptNumber, orderId]
  );

  return result.rowCount > 0;
}
```

Two things happen:

1. The `AND payment_status != 'success'` clause makes the UPDATE a no-op if the order is already paid.
2. `RETURNING id` tells us whether a row was actually modified. `rowCount === 0` means "already processed"; `rowCount === 1` means "we did the transition".

Now the check and the write happen in a single atomic SQL statement. No race window.

```javascript
// In the callback handler:
const didTransition = await markPaid(order.id, parsed.mpesaReceiptNumber);
if (didTransition) {
  // Fire side effects (notifications)
  orderNotifications.sendOrderConfirmation(order.id).catch(console.error);
}
// If !didTransition, someone else already processed this. Skip.
```

This pattern is called **optimistic concurrency control**. Use it anywhere a webhook handler transitions a state.

---

## Fix 2: The Outbox Pattern

You can still crash between the state change and the side effect. Database says `paid`, WhatsApp was never sent, retry finds `paid` and skips the send. The customer is angry.

The fix: **write the side effect to an "outbox" table in the same transaction as the state change**, and have a separate worker process the outbox.

```sql
CREATE TABLE outbox (
  id BIGSERIAL PRIMARY KEY,
  event_type TEXT NOT NULL,
  payload JSONB NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  processed_at TIMESTAMPTZ,
  attempts INTEGER NOT NULL DEFAULT 0
);
CREATE INDEX idx_outbox_unprocessed ON outbox(created_at) WHERE processed_at IS NULL;
```

In the handler:

```javascript
const client = await pool.connect();
try {
  await client.query("BEGIN");

  const updated = await client.query(
    `UPDATE orders
     SET payment_status = 'success', status = 'paid', updated_at = NOW()
     WHERE id = $1 AND payment_status != 'success'
     RETURNING id`,
    [orderId]
  );

  if (updated.rowCount === 0) {
    await client.query("ROLLBACK");
    return; // already processed
  }

  // Write the side effect to the outbox IN THE SAME TRANSACTION
  await client.query(
    `INSERT INTO outbox (event_type, payload) VALUES ($1, $2)`,
    ["order.paid", JSON.stringify({ orderId })]
  );

  await client.query("COMMIT");
} catch (err) {
  await client.query("ROLLBACK");
  throw err;
}
```

A separate worker polls the outbox every few seconds:

```javascript
// server/workers/outbox.js
const { query } = require("../config/db");
const orderNotifications = require("../services/orderNotifications.service");

async function processOutbox() {
  const { rows } = await query(
    `SELECT id, event_type, payload FROM outbox
     WHERE processed_at IS NULL AND attempts < 5
     ORDER BY created_at ASC
     LIMIT 10`
  );

  for (const row of rows) {
    try {
      if (row.event_type === "order.paid") {
        await orderNotifications.sendOrderConfirmation(row.payload.orderId);
      }
      // ... other event types

      await query(
        `UPDATE outbox SET processed_at = NOW() WHERE id = $1`,
        [row.id]
      );
    } catch (err) {
      console.error(`outbox ${row.id} failed:`, err);
      await query(
        `UPDATE outbox SET attempts = attempts + 1 WHERE id = $1`,
        [row.id]
      );
    }
  }
}

// Run every 5 seconds
setInterval(processOutbox, 5000);
module.exports = { processOutbox };
```

Now the flow is bullet-proof:

1. Callback arrives. Transaction marks `paid` and writes to outbox. Crash here? Nothing committed. Retry re-runs cleanly.
2. Transaction commits. Both the state change and the outbox row exist. Crash here? On restart the worker picks up the outbox row and sends the WhatsApp. Crash during send? Attempts counter increments, retries next poll.
3. Worker sends the message and marks the outbox row processed. Done.

This is the **transactional outbox pattern** and it is how Uber, Airbnb, and every real payment system handle side effects. Week 23 we replace the polling worker with BullMQ for scale, but the outbox table stays.

---

## Fix 3: Out-Of-Order Events

A callback for `delivered` arriving before the callback for `paid` is weird but possible -- providers and networks do strange things. Your transition table refuses `pending -> delivered`, so the late callback errors out.

The fix: **store events, not transitions**. Keep an append-only log of every state-relevant event. Your "current state" is a derived value from the events in timestamp order.

```sql
CREATE TABLE order_events (
  id BIGSERIAL PRIMARY KEY,
  order_id UUID NOT NULL REFERENCES orders(id),
  event_type TEXT NOT NULL,
  occurred_at TIMESTAMPTZ NOT NULL,
  metadata JSONB
);
CREATE INDEX idx_order_events_order ON order_events(order_id, occurred_at);
```

When a callback arrives, insert an event with the provider's timestamp, then recompute the order's current status from events:

```javascript
async function recomputeOrderStatus(orderId) {
  const { rows: events } = await query(
    `SELECT event_type, occurred_at FROM order_events
     WHERE order_id = $1 ORDER BY occurred_at DESC`,
    [orderId]
  );

  // Simple rule: the latest event determines the status.
  // More nuanced: use a priority order (delivered > shipped > paid > pending).
  const latest = events[0];
  if (!latest) return "pending";

  const MAP = {
    "payment_initiated": "pending",
    "payment_succeeded": "paid",
    "shipped": "shipped",
    "delivered": "delivered",
    "cancelled": "cancelled",
  };

  return MAP[latest.event_type] || "pending";
}
```

With events, an out-of-order `delivered` and `paid` both land in the log. Recomputation sorts them, sees `delivered` as the latest, and returns `delivered`. Order of arrival no longer matters.

This is called **event sourcing**. It is a significant architecture choice and you do not have to rewrite everything around it -- for the Marathon, keep the `status` column as a cache of the recomputed value and treat the events table as the source of truth for audit and recovery. Real companies do something similar; Stripe itself is event-sourced internally.

---

## Idempotency Keys (The Stripe Pattern)

For **outgoing** requests (you call a provider's API), send an `Idempotency-Key` header. Stripe supports this natively:

```javascript
const session = await stripe.checkout.sessions.create(
  { /* session data */ },
  { idempotencyKey: `order-${orderId}` }
);
```

If you call this twice with the same key, Stripe returns the same session both times -- no double charge. The retry case is solved at the API boundary.

Daraja does not support this directly. For M-Pesa you generate a unique `AccountReference` per order and Daraja deduplicates by that. For Airtel you use the `transaction.id` field.

The pattern:
- Generate a per-order unique key once, store it, reuse it on retries.
- Pass it to the provider as whatever they call their idempotency field.
- If you ever retry, use the same key.

**Never** generate a fresh key per call. That defeats the whole point.

---

## Testing Idempotency

Write a small script that POSTs the same callback body to your endpoint 10 times in parallel and verifies:

1. The order ends up in the correct state.
2. The outbox has exactly one row for the event.
3. The WhatsApp was sent exactly once.

```javascript
// scripts/test-idempotency.js
const fetch = require("node-fetch");
const body = { /* real M-Pesa callback payload */ };

async function main() {
  const promises = Array.from({ length: 10 }, () =>
    fetch("http://localhost:5000/api/payments/callback/mpesa", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(body),
    })
  );
  await Promise.all(promises);

  // Then query the database:
  const { rows } = await query("SELECT * FROM outbox WHERE payload->>'orderId' = $1", [orderId]);
  console.assert(rows.length === 1, `Expected 1 outbox row, got ${rows.length}`);
}

main();
```

If the assertion fires, your idempotency is broken. Fix it before shipping.

---

## Checkpoint

1. The M-Pesa callback handler uses atomic check-and-set (`WHERE payment_status != 'success'`).
2. The Stripe webhook writes side effects to the outbox in the same transaction as the state change.
3. A worker process picks up outbox rows and delivers them.
4. The parallel-callback test finds only one outbox row and one WhatsApp send.
5. Posting a callback for an already-paid order does not add a new outbox row.
6. An out-of-order test (shipped-before-paid) ends with the order in the correct final state.
7. Stripe session creation uses an `idempotencyKey` derived from the order id.

Commit:

```bash
git add .
git commit -m "feat: atomic state transitions outbox pattern and idempotency keys"
```

---

## What You Learned

- Atomic `UPDATE ... WHERE not-already` replaces check-then-update races.
- The transactional outbox pattern gives you reliable side effects after state changes.
- Event sourcing via an append-only log fixes out-of-order callbacks.
- Idempotency keys on outgoing calls prevent accidental duplicates at the provider side.
- Test idempotency by firing parallel duplicates and counting outcomes.

Tomorrow is refunds -- the flip side of payments, and surprisingly tricky on each provider.
