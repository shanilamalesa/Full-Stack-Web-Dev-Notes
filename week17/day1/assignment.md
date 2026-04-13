# Week 17 - Day 1 Assignment

## Title
Stripe Foundations -- Add International Card Payments

## Overview
Week 17 adds a second and third payment provider. Today is Stripe: create a test account, install the Stripe Node SDK, and add a Checkout Session route that a customer can use to pay with a card.

## Learning Objectives Assessed
- Sign up for a Stripe test account
- Install and configure the Stripe SDK
- Create a Checkout Session via a Server Action
- Handle Stripe's redirect flow
- Understand the difference between Payment Intents and Checkout Sessions

## Prerequisites
- Week 16 completed

## AI Usage Rules

**Ratio this week:** 30% manual / 70% AI
**Habit:** Provider docs first, implementation second. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Stripe SDK boilerplate after you read the docs.
- **NOT ALLOWED FOR:** Copying any money line without reading what it does.
- **AUDIT REQUIRED:** Yes. Money rule still applies.

## Tasks

### Task 1: Stripe test account

**What to do:**
1. Go to https://dashboard.stripe.com/register
2. Sign up. Skip the business info (test mode is fine).
3. Get your test API keys from Developers > API keys:
   - `STRIPE_SECRET_KEY=sk_test_...`
   - `STRIPE_PUBLISHABLE_KEY=pk_test_...`
4. Add to `.env.local`.

**Expected output:**
Both keys in your env. NEVER commit them.

### Task 2: Install Stripe SDK

**What to do:**
```bash
npm install stripe
```

Create `lib/stripe.js`:

```javascript
import Stripe from "stripe";
export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: "2023-10-16",
});
```

**Expected output:**
`lib/stripe.js` in place. Import works.

### Task 3: Checkout Session Server Action

**What to do:**
Create `app/checkout/stripeAction.js`:

```javascript
"use server";

import { stripe } from "@/lib/stripe";
import pool from "@/lib/db";

export async function createStripeCheckoutSession(orderId) {
  const { rows } = await pool.query("SELECT total_cents FROM orders WHERE id = $1", [orderId]);
  const order = rows[0];
  if (!order) return { error: "Order not found" };

  const session = await stripe.checkout.sessions.create({
    mode: "payment",
    payment_method_types: ["card"],
    line_items: [
      {
        price_data: {
          currency: "kes",
          product_data: { name: `Order ${orderId.slice(0, 8)}` },
          unit_amount: order.total_cents,
        },
        quantity: 1,
      },
    ],
    success_url: `${process.env.APP_URL}/orders/${orderId}?stripe_success=1`,
    cancel_url: `${process.env.APP_URL}/checkout?cancelled=1`,
    metadata: { order_id: orderId },
  });

  await pool.query(
    "UPDATE orders SET stripe_session_id = $1 WHERE id = $2",
    [session.id, orderId]
  );

  return { url: session.url };
}
```

Add `stripe_session_id TEXT` column to `orders`. Add `APP_URL` to env.

**Expected output:**
Server Action creates a real Stripe Checkout Session.

### Task 4: Wire into payment step

**What to do:**
In your checkout PaymentStep, when user picks "Card", call the Stripe action and redirect to the session URL:

```jsx
if (paymentMethod === "card") {
  const result = await createStripeCheckoutSession(orderId);
  if (result.url) window.location.href = result.url;
  else dispatch({ type: "ERROR", message: result.error });
}
```

**Expected output:**
Picking Card redirects to Stripe's hosted checkout page.

### Task 5: Test with a Stripe test card

**What to do:**
Stripe's dashboard shows test cards. Use `4242 4242 4242 4242`, any future expiry, any 3-digit CVC, any ZIP.

Complete the payment. Stripe redirects you to your `success_url`. Verify the URL contains `stripe_success=1`.

**Expected output:**
Screenshot `day1-stripe-success.png` of the redirected URL.

## Stretch Goals (Optional - Extra Credit)

- Test a declined card (`4000 0000 0000 0002`) and see the error flow.
- Add a Stripe-specific confirmation message.
- Use the Stripe Elements approach instead of hosted Checkout.

## Submission Requirements

- **What to submit:** Repo with Stripe action, screenshots, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Stripe test account + keys in env | 10 | Keys set. Not committed. |
| Stripe SDK installed | 10 | Import works. |
| Checkout Session Server Action | 35 | Creates real session. Persists session id. |
| Wired to payment step | 20 | Card redirects to Stripe. |
| Test payment completed | 15 | Screenshot confirms. |
| Clean commits | 10 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using `STRIPE_PUBLISHABLE_KEY` server-side.** That key is for client use. Use the secret key on the server.
- **Forgetting to set the currency.** KES is correct for Kenyan shops. `currency: "kes"`.
- **Using whole shillings instead of cents.** Stripe uses the smallest unit like Daraja. 150000 = KES 1500.

## Resources

- Day 1 reading: [Stripe Foundations.md](./Stripe%20Foundations.md)
- Week 17 AI boundaries: [../ai.md](../ai.md)
- Stripe Checkout docs: https://stripe.com/docs/payments/checkout
