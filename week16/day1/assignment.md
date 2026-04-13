# Week 16 - Day 1 Assignment

## Title
Wire M-Pesa STK Push Into The Next.js Shop

## Overview
Week 16 ships Project 3: a working e-commerce shop with real M-Pesa payments and WhatsApp order confirmations. Today you connect your Week 10 M-Pesa code to the Next.js shop's checkout flow via a Server Action that initiates the STK Push.

## Learning Objectives Assessed
- Call Daraja from a Next.js Server Action
- Wire M-Pesa as a payment option in the checkout state machine
- Return a checkout-request-id for polling
- Keep money code manual despite the overall high AI ratio

## Prerequisites
- Week 15 completed (Next.js shop with checkout)
- Week 10 backend M-Pesa code available to copy

## AI Usage Rules

**Ratio this week:** 30% manual / 70% AI
**Habit:** AI as integration engineer -- AI wires the ports you designed. Money rule still applies. See [../ai.md](../ai.md).

- **ALLOWED FOR:** UI scaffolding, repeated patterns.
- **NOT ALLOWED FOR:** Any line that touches amounts, references, or idempotency. Money rule is permanent.
- **AUDIT REQUIRED:** Yes. Money audit continues.

## Tasks

### Task 1: Copy M-Pesa service into Next.js

**What to do:**
Create `lib/mpesa.js` in your Next.js project. Port the `getToken` and `initiateStkPush` functions from Week 10. Hand-type every money-related line. Add the env variables to `.env.local`.

**Expected output:**
`lib/mpesa.js` matches Week 10 logic. Uses the same env pattern.

### Task 2: Server Action to initiate STK push

**What to do:**
Create `app/checkout/mpesaAction.js`:

```javascript
"use server";

import { initiateStkPush } from "@/lib/mpesa";
import pool from "@/lib/db";

export async function initiateMpesaPayment(orderId, phone, amountCents) {
  // Validate -- manual
  if (!phone || !/^2547\d{8}$/.test(phone)) return { error: "Invalid phone" };
  if (!amountCents || amountCents < 100) return { error: "Invalid amount" };

  try {
    const result = await initiateStkPush({
      phone,
      amount: Math.ceil(amountCents / 100),
      accountRef: orderId.slice(0, 12),
      description: `Order ${orderId.slice(0, 8)}`,
    });

    // Persist the checkout request id on the order
    await pool.query(
      "UPDATE orders SET mpesa_checkout_id = $1 WHERE id = $2",
      [result.CheckoutRequestID, orderId]
    );

    return { checkoutRequestId: result.CheckoutRequestID };
  } catch (err) {
    console.error(err);
    return { error: "Payment initiation failed" };
  }
}
```

Add `mpesa_checkout_id TEXT` column to your orders table.

**Expected output:**
Server Action created. Hand-typed. Includes validation.

### Task 3: Wire into the checkout flow

**What to do:**
In your Week 15 PaymentStep component, when the user picks "M-Pesa", call the Server Action (instead of the fake success):

```jsx
if (state.paymentMethod === "mpesa") {
  const result = await initiateMpesaPayment(orderId, state.customer.phone, total);
  if (result.error) dispatch({ type: "ERROR", message: result.error });
  else dispatch({ type: "AWAIT_PAYMENT", checkoutRequestId: result.checkoutRequestId });
}
```

Add `AWAIT_PAYMENT` state to your checkout reducer. For now it just shows "Check your phone for the M-Pesa prompt".

**Expected output:**
Picking M-Pesa triggers a real STK Push to your test phone. Screenshot `day1-stk.png`.

### Task 4: Test happy path

**What to do:**
Run through the full flow with sandbox credentials:
1. Add product to cart
2. Checkout -> info -> payment -> M-Pesa
3. STK prompt on phone
4. Enter PIN

Verify the order is created and `mpesa_checkout_id` is set.

**Expected output:**
Full flow works end to end in the sandbox.

### Task 5: Money audit update

**What to do:**
In `AI_AUDIT.md`, update the money code audit to include:
- `lib/mpesa.js` -- hand-written, copied from Week 10
- `app/checkout/mpesaAction.js` -- hand-written, no AI on validation or amounts
- Server Action wiring -- AI helped with the dispatch integration, but money calculations remain manual

**Expected output:**
Audit updated.

## Stretch Goals (Optional - Extra Credit)

- Handle the error case where Daraja times out (no response within 60s).
- Add a retry button on errors.
- Log every STK initiation to a `payment_attempts` table.

## Submission Requirements

- **What to submit:** Repo, `lib/mpesa.js`, action, updated checkout page, screenshots, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| lib/mpesa.js ported correctly | 20 | Token + STK initiator working. |
| Server Action with validation | 25 | Hand-typed, validates phone and amount. |
| Checkout wired to STK | 20 | M-Pesa path triggers real STK. |
| Full sandbox test | 20 | End-to-end works. |
| Money audit updated | 10 | Every money line classified. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Returning the amount from the client.** The client can be tampered with. Always compute the amount on the server from the cart state (which you can trust because you validated the products before).
- **Forgetting to persist the checkout id.** You need it for the callback reconciliation.
- **Letting AI touch the validation code.** Money rule.

## Resources

- Day 1 reading: [M-Pesa Inside the Shop.md](./M-Pesa%20Inside%20the%20Shop.md)
- Week 16 AI boundaries: [../ai.md](../ai.md)
- Week 10 M-Pesa assignments for reference.
