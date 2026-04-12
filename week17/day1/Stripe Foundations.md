# Week 17, Day 1: Stripe Foundations

By the end of today, your shop accepts international card payments through Stripe. A customer outside Kenya can land on your product page, click Pay, enter card details in a Stripe-hosted checkout, and their card gets charged -- with webhooks updating your orders table. Stripe Elements, Stripe Checkout, test cards, and the same callback pattern you learned for M-Pesa.

**Prior-week concepts you will use today:**
- The payments service pattern from Week 16 (same shape, new provider)
- Webhook handling with idempotency (Week 16, Day 2)
- Next.js Server Actions (Week 15, Day 3)
- Orders schema with `payment_method` already in place (Week 15, Day 3)

**Estimated time:** 3-4 hours

---

## The Week Ahead

| Day | Focus |
|---|---|
| Day 1 (today) | Stripe Checkout integration for card payments. Webhook for `payment_intent.succeeded`. |
| Day 2 | Airtel Money integration. Completing the three-provider picture. |
| Day 3 | A unified Payments service: one interface, three implementations. |
| Day 4 | Provider selection UI and regional routing. |
| Day 5 | Recap. |
| Weekend | A shop that gracefully accepts M-Pesa, Airtel, and Stripe depending on the customer. |

---

## Why Add Stripe At All

You already have M-Pesa. For a purely Kenyan shop, that is enough -- 97% of Kenyan customers prefer mobile money. But two cases put Stripe on the critical path:

1. **Diaspora customers.** A Kenyan living in Dubai wants to order a gift for family in Nakuru. Their phone does not do M-Pesa; their Visa does.
2. **Tourists and businesses.** A Nairobi hotel accepting international bookings needs cards. A consulting firm billing clients in Europe needs a way to invoice.

Stripe is the global standard for card payments. It also handles Apple Pay, Google Pay, SEPA, and dozens of regional methods. For a small Kenyan SME the single use case is "accept a Visa from a non-Kenyan". That is enough to matter.

The risk of adding Stripe is complexity. You now have two payment providers, two webhook shapes, two token systems, two test suites. Day 3 we unify them into one interface so the shop code does not have to care; today we just add the second provider.

---

## Setting Up a Stripe Test Account

Go to https://stripe.com and sign up. You do not need to fill in business details to use test mode -- skip straight to the dashboard. Stripe's test mode is free and gives you:

- A dashboard with test API keys
- Test card numbers that simulate every possible outcome
- A webhook forwarding CLI tool

Get your test keys from **Developers -> API keys**:

- **Publishable key**: `pk_test_...` -- safe to expose in the browser.
- **Secret key**: `sk_test_...` -- keep out of client-side code and git.

Add to `shop/.env.local`:

```env
STRIPE_SECRET_KEY=sk_test_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
```

Install the Stripe Node library and the React Stripe bindings:

```bash
cd shop
npm install stripe @stripe/stripe-js
```

---

## Stripe Checkout vs Stripe Elements

Stripe offers two integration modes:

- **Stripe Checkout** -- Stripe hosts the entire payment page. You redirect the customer to `checkout.stripe.com/c/...`, they pay, Stripe redirects them back. You implement this in about 20 lines of code.
- **Stripe Elements** -- Stripe provides embeddable card input components. The customer never leaves your site. More control, more work.

For Project 3 we use **Stripe Checkout**. Two reasons:

1. **PCI compliance is free.** Because card data never touches your server, you are automatically in the lowest PCI compliance tier. Elements puts you one step higher and requires annual paperwork.
2. **Simpler code.** Less surface area, fewer things to debug. The UX cost is one redirect away from your site and one redirect back -- most customers do not notice.

Elements makes sense when you need a fully custom checkout experience or you cannot afford the redirect. We have neither constraint.

---

## The Server Action

Create `app/actions/createStripeCheckout.js`:

```javascript
// app/actions/createStripeCheckout.js
"use server";

import Stripe from "stripe";
import { query, pool } from "@/lib/db";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export async function createStripeCheckout({ customer, items }) {
  // Validation (same as createOrder, minus paymentMethod since it is Stripe by definition)
  if (!customer?.name || !customer?.phone) return { error: "Missing customer info" };
  if (!items || items.length === 0) return { error: "Cart is empty" };

  const client = await pool.connect();
  try {
    await client.query("BEGIN");

    // Fetch current prices + stock
    const ids = items.map((i) => i.id);
    const { rows: products } = await client.query(
      "SELECT id, name, price_cents, stock FROM products WHERE id = ANY($1) FOR UPDATE",
      [ids]
    );
    const productMap = Object.fromEntries(products.map((p) => [p.id, p]));

    let subtotalCents = 0;
    const lineItems = [];
    for (const item of items) {
      const p = productMap[item.id];
      if (!p) throw new Error("Product not found");
      if (p.stock < item.quantity) throw new Error(`Out of stock: ${p.name}`);
      subtotalCents += p.price_cents * item.quantity;
      lineItems.push({
        price_data: {
          currency: "kes",
          product_data: { name: p.name },
          unit_amount: p.price_cents,
        },
        quantity: item.quantity,
      });
    }

    // Insert the order (same as Week 15)
    const { rows: orderRows } = await client.query(
      `INSERT INTO orders (customer_name, customer_phone, delivery_address, subtotal_cents, payment_method, status)
       VALUES ($1, $2, $3, $4, 'stripe', 'pending')
       RETURNING id`,
      [customer.name, customer.phone, customer.address, subtotalCents]
    );
    const orderId = orderRows[0].id;

    // Insert order items
    for (const item of items) {
      const p = productMap[item.id];
      await client.query(
        `INSERT INTO order_items (order_id, product_id, quantity, price_cents_at_purchase)
         VALUES ($1, $2, $3, $4)`,
        [orderId, item.id, item.quantity, p.price_cents]
      );
    }

    // Stock deduction
    for (const item of items) {
      await client.query(
        `UPDATE products SET stock = stock - $1 WHERE id = $2`,
        [item.quantity, item.id]
      );
    }

    // Create the Stripe Checkout session
    const session = await stripe.checkout.sessions.create({
      mode: "payment",
      payment_method_types: ["card"],
      line_items: lineItems,
      customer_email: customer.email || undefined,
      success_url: `${process.env.NEXT_PUBLIC_BASE_URL}/checkout/confirmed?session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: `${process.env.NEXT_PUBLIC_BASE_URL}/checkout/cancelled?order=${orderId}`,
      metadata: {
        order_id: orderId,
      },
    });

    // Save the session id for lookup
    await client.query(
      "UPDATE orders SET stripe_session_id = $1 WHERE id = $2",
      [session.id, orderId]
    );

    await client.query("COMMIT");

    return { orderId, url: session.url };
  } catch (err) {
    await client.query("ROLLBACK");
    console.error("Stripe checkout failed:", err);
    return { error: err.message || "Checkout failed" };
  } finally {
    client.release();
  }
}
```

Add the column:

```sql
ALTER TABLE orders ADD COLUMN stripe_session_id TEXT;
CREATE INDEX idx_orders_stripe_session ON orders(stripe_session_id);
```

Two things are new compared to Week 15 / Week 16.

**`line_items` with `price_data`.** Stripe lets you define prices inline without creating Product and Price objects first. For a dynamic shop this is simpler than mirroring every product into Stripe. The downside is you lose Stripe's reporting by product -- for a small shop that is fine.

**`metadata.order_id`.** Stripe lets you attach arbitrary key-value metadata to a session. We attach the order id so the webhook can look up the order. This is the `CheckoutRequestID` equivalent for Stripe.

### The success redirect

When the customer pays, Stripe redirects them to `/checkout/confirmed?session_id=cs_test_abc`. The success URL contains the session id as a query param. We could verify the payment client-side using this id -- but **do not**. Client-side verification is trivially bypassable. The source of truth is the webhook. Use the success page for "show the customer a nice page", use the webhook for "update the database".

---

## The Webhook

Stripe webhooks are a POST to a URL you register. The request body is JSON; the request headers include a signature you must verify. Unlike M-Pesa, Stripe has a proper HMAC signature system and you absolutely must verify it.

Create `app/api/webhooks/stripe/route.js`:

```javascript
// app/api/webhooks/stripe/route.js
import Stripe from "stripe";
import { query } from "@/lib/db";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);
const WEBHOOK_SECRET = process.env.STRIPE_WEBHOOK_SECRET;

export async function POST(req) {
  const signature = req.headers.get("stripe-signature");
  const rawBody = await req.text();

  let event;
  try {
    event = stripe.webhooks.constructEvent(rawBody, signature, WEBHOOK_SECRET);
  } catch (err) {
    console.error("Invalid Stripe signature:", err);
    return Response.json({ error: "Invalid signature" }, { status: 400 });
  }

  // Idempotency: Stripe sends event.id; use it as a dedup key
  if (await alreadyProcessed(event.id)) {
    return Response.json({ received: true });
  }

  try {
    switch (event.type) {
      case "checkout.session.completed": {
        const session = event.data.object;
        const orderId = session.metadata?.order_id;
        if (orderId) {
          await query(
            `UPDATE orders
             SET payment_status = 'success',
                 status = 'paid',
                 updated_at = NOW()
             WHERE id = $1 AND payment_method = 'stripe'`,
            [orderId]
          );
          // Fire-and-forget WhatsApp notification
          fetch(`${process.env.CRM_SERVER_URL}/api/orders/${orderId}/notify`, {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ event: "paid" }),
          }).catch(() => {});
        }
        break;
      }

      case "checkout.session.expired": {
        const session = event.data.object;
        const orderId = session.metadata?.order_id;
        if (orderId) {
          await query(
            `UPDATE orders SET payment_status = 'cancelled' WHERE id = $1`,
            [orderId]
          );
        }
        break;
      }

      case "payment_intent.payment_failed": {
        // Log for observability; customer can retry
        console.log("Payment failed:", event.data.object.id);
        break;
      }

      default:
        // Ignore unhandled events
        break;
    }

    await markProcessed(event.id);
    return Response.json({ received: true });
  } catch (err) {
    console.error("Webhook processing error:", err);
    return Response.json({ error: "Processing failed" }, { status: 500 });
  }
}

async function alreadyProcessed(eventId) {
  const { rows } = await query(
    "SELECT 1 FROM webhook_events WHERE id = $1",
    [eventId]
  );
  return rows.length > 0;
}

async function markProcessed(eventId) {
  await query(
    "INSERT INTO webhook_events (id, received_at) VALUES ($1, NOW()) ON CONFLICT DO NOTHING",
    [eventId]
  );
}
```

And the table:

```sql
CREATE TABLE webhook_events (
  id TEXT PRIMARY KEY,
  received_at TIMESTAMPTZ NOT NULL
);
```

Three things worth reading carefully.

**`stripe.webhooks.constructEvent`** verifies the signature and parses the body in one call. If the signature is wrong, it throws. Never process a webhook whose signature you have not verified -- anyone could POST fake events and flip orders to `paid`.

**`event.id` deduplication.** Stripe guarantees that `event.id` is unique per event but may retry the same event multiple times (network issues on your side). The `webhook_events` table is a tiny audit log that gives us idempotency for free. Insert-or-skip on conflict, return early if we have seen this event before.

**The switch statement.** Stripe sends hundreds of event types; we only care about three. The `default` branch silently ignores the rest. Fine and safe -- a future version of Stripe will add new event types and our code keeps working.

---

## Testing The Webhook Locally

Stripe can forward live webhooks to your localhost via the Stripe CLI. Install it:

```bash
# Mac
brew install stripe/stripe-cli/stripe

# Linux
# Download from https://github.com/stripe/stripe-cli/releases
```

Log in once:

```bash
stripe login
```

Forward webhooks to your Next.js dev server:

```bash
stripe listen --forward-to http://localhost:3000/api/webhooks/stripe
```

The command prints a webhook signing secret (`whsec_...`). Copy it into `.env.local` as `STRIPE_WEBHOOK_SECRET`. Restart Next.js.

Now run a test checkout:

1. Go to `/checkout/payment`, select Stripe, click Pay.
2. You are redirected to `checkout.stripe.com`.
3. Enter the test card `4242 4242 4242 4242`, any future expiry, any CVC.
4. Submit. Stripe processes, redirects to `/checkout/confirmed`.
5. Back in the Stripe CLI terminal, you see the forwarded events: `checkout.session.completed`, `payment_intent.succeeded`, etc.
6. Your Next.js server logs processing each event.
7. `psql` shows the order flipped to `paid`.

Test cards to know:
- `4242 4242 4242 4242` -- always succeeds.
- `4000 0000 0000 0002` -- declined (generic).
- `4000 0025 0000 3155` -- requires 3D Secure authentication.
- `4000 0000 0000 9995` -- insufficient funds.

Run one of each. Your webhook should handle them correctly (success for the first, no update for the rest except eventually expiring).

---

## Production Webhook Setup

When you deploy, you register the production webhook URL with Stripe via the dashboard:

1. **Developers -> Webhooks -> Add endpoint**.
2. URL: `https://yourshop.co.ke/api/webhooks/stripe`.
3. Events to listen for: `checkout.session.completed`, `checkout.session.expired`, `payment_intent.payment_failed`.
4. Stripe generates a new signing secret -- use this in production.

Never reuse the CLI-forwarded signing secret in production. It is scoped to your CLI session.

---

## Checkpoint

1. A test checkout with card `4242...` on the Stripe sandbox completes and the order flips to `paid`.
2. The Stripe CLI log shows the `checkout.session.completed` event being delivered.
3. The `webhook_events` table has one row per event you have processed.
4. POSTing the same webhook twice (copy a payload, curl it) does not double-update the order.
5. POSTing a webhook with a bad `stripe-signature` header returns 400 and does not write to the database.
6. A declined card (`4000 0000 0000 0002`) leaves the order in `payment_status = 'not_initiated'` or `'cancelled'` -- not `paid`.
7. The `stripe_session_id` column is populated on the order row.
8. The customer sees the Stripe-hosted checkout page, not an embedded form. Confirming that is one of the design decisions paying off.

Commit:

```bash
git add .
git commit -m "feat: stripe checkout for international card payments"
```

---

## What You Learned

- Stripe Checkout hosts the payment UI; Elements embeds it.
- PCI compliance is almost free when card data never touches your server.
- `metadata.order_id` is the Stripe equivalent of `CheckoutRequestID`.
- Webhook signatures must be verified; `constructEvent` does it for you.
- `event.id` deduplication gives you free idempotency via a small audit table.
- The Stripe CLI forwards live webhooks to localhost for dev.

Tomorrow we add Airtel Money, and by Wednesday the shop speaks three payment languages.
