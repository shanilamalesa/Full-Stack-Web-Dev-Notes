# Week 17 - Day 3 Assignment

## Title
A Unified Payments Interface -- One Function, Three Providers

## Overview
Today you wrap your three payment providers (M-Pesa, Stripe, Airtel) behind a single interface. The checkout page calls one function, which routes to the right provider. This is the seed of the reusable Payments Service you will publish in Week 19.

## Learning Objectives Assessed
- Design a common shape across three different provider APIs
- Hide provider-specific quirks behind one interface
- Use a strategy pattern to pick the provider
- Return a consistent response shape

## Prerequisites
- Days 1-2 completed

## AI Usage Rules

**Ratio:** 30/70. **Habit:** Provider docs first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Adapter scaffolding.
- **NOT ALLOWED FOR:** Designing the shared interface.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Design the interface

**What to do:**
Before writing code, in `interface-design.md`, define the shared interface:

```typescript
// Conceptual. JS comments for now.
type InitiateRequest = {
  orderId: string;
  amountCents: number;
  phone?: string;          // required for mobile money
  email?: string;          // required for card
  provider: "mpesa" | "airtel" | "stripe";
};

type InitiateResponse = {
  success: boolean;
  reference?: string;       // provider reference (checkout id, session id, etc)
  redirectUrl?: string;     // for providers that redirect (Stripe)
  error?: string;
};
```

Write the design in your own words. No AI.

**Expected output:**
`interface-design.md` committed.

### Task 2: Create the adapters

**What to do:**
Create `lib/payments/index.js`:

```javascript
import { initiateMpesa } from "./mpesa";
import { initiateStripe } from "./stripe";
import { initiateAirtel } from "./airtel";

export async function initiatePayment({ provider, orderId, amountCents, phone, email }) {
  switch (provider) {
    case "mpesa":
      return initiateMpesa({ orderId, amountCents, phone });
    case "stripe":
      return initiateStripe({ orderId, amountCents, email });
    case "airtel":
      return initiateAirtel({ orderId, amountCents, phone });
    default:
      return { success: false, error: `Unknown provider: ${provider}` };
  }
}
```

Each adapter wraps its provider and returns the common shape.

**Expected output:**
Unified interface in place. Three adapters.

### Task 3: Normalise responses

**What to do:**
Each adapter must return `{ success, reference, redirectUrl?, error? }`. For M-Pesa and Airtel, `reference` is the checkout request id. For Stripe, it is the session id and `redirectUrl` is set.

```javascript
// lib/payments/stripe.js
export async function initiateStripe({ orderId, amountCents, email }) {
  try {
    const session = await stripe.checkout.sessions.create({ /* ... */ });
    return {
      success: true,
      reference: session.id,
      redirectUrl: session.url,
    };
  } catch (err) {
    return { success: false, error: err.message };
  }
}
```

**Expected output:**
All three adapters return the same shape.

### Task 4: Refactor the checkout page

**What to do:**
Replace the per-provider switch statements in your checkout page with one call:

```jsx
const result = await initiatePayment({
  provider: state.paymentMethod,
  orderId,
  amountCents: total,
  phone: state.customer.phone,
  email: state.customer.email,
});

if (result.redirectUrl) window.location.href = result.redirectUrl;
else if (result.success) dispatch({ type: "AWAIT_PAYMENT", reference: result.reference });
else dispatch({ type: "ERROR", message: result.error });
```

**Expected output:**
Checkout has one code path regardless of provider.

### Task 5: Test all three

**What to do:**
Run a full test for each provider. Verify each one reaches a success state (or redirect) via the same checkout component.

**Expected output:**
Three screenshots: `day3-mpesa.png`, `day3-stripe.png`, `day3-airtel.png`.

## Stretch Goals (Optional - Extra Credit)

- Add a `capture` interface for asynchronous captures (some payments are authorised first, captured later).
- Add provider health checks.
- Add a fallback: if M-Pesa fails, offer Stripe.

## Submission Requirements

- **What to submit:** Repo, `interface-design.md`, adapters, screenshots, `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Interface design written first | 20 | Design doc committed before code. |
| Three adapters with common shape | 30 | Each returns `{ success, reference, redirectUrl?, error? }`. |
| Unified initiatePayment function | 20 | One function, one switch. |
| Checkout uses the unified function | 15 | No more per-provider branches in the UI. |
| All three tested | 10 | Three screenshots. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Letting adapter-specific fields leak out.** The UI should not know what `CheckoutRequestID` vs `session_id` means.
- **Writing the unified function before the adapters.** Build the concrete cases first, then extract.
- **Skipping the design doc.** The doc is the cheap way to think through the shape before committing to code.

## Resources

- Day 3 reading: [A Unified Payments Interface.md](./A%20Unified%20Payments%20Interface.md)
- Week 17 AI boundaries: [../ai.md](../ai.md)
