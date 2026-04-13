# Week 15 - Day 2 Assignment

## Title
Checkout State Machine -- Customer Info, Payment, Confirmation

## Overview
Yesterday you built the cart. Today you extend it into a full checkout flow: a second state machine that transitions `cart -> info -> payment -> confirmed`. You will sketch it on paper first, then implement a multi-step checkout form.

## Learning Objectives Assessed
- Design a multi-step flow as a state machine on paper
- Implement the state machine with useReducer
- Build multi-step forms that preserve data between steps
- Handle "back" navigation gracefully

## Prerequisites
- Day 1 completed

## AI Usage Rules

**Ratio:** 35/65. **Habit:** Paper first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Form UI after you sketched the states.
- **NOT ALLOWED FOR:** Designing the states.
- **AUDIT REQUIRED:** Yes. Second sketch required.

## Tasks

### Task 1: Sketch the checkout

**What to do:**
On paper, draw:
- States: `cart`, `info`, `payment`, `processing`, `confirmed`, `error`
- Transitions between them
- Data that lives in context: customer info, payment method, order id

Photograph and commit as `assets/day2-checkout-sketch.jpg`.

**Expected output:**
Second sketch committed.

### Task 2: Checkout reducer

**What to do:**
Create `app/checkout/checkoutReducer.js`:

```javascript
export const initialState = {
  step: "cart",
  customer: { name: "", email: "", phone: "", address: "" },
  paymentMethod: null,
  orderId: null,
  error: null,
};

export function checkoutReducer(state, action) {
  switch (action.type) {
    case "NEXT":
      if (state.step === "cart") return { ...state, step: "info" };
      if (state.step === "info") return { ...state, step: "payment" };
      if (state.step === "payment") return { ...state, step: "processing" };
      return state;
    case "BACK":
      if (state.step === "info") return { ...state, step: "cart" };
      if (state.step === "payment") return { ...state, step: "info" };
      return state;
    case "SET_CUSTOMER":
      return { ...state, customer: { ...state.customer, ...action.payload } };
    case "SET_PAYMENT":
      return { ...state, paymentMethod: action.payload };
    case "SUCCESS":
      return { ...state, step: "confirmed", orderId: action.orderId };
    case "ERROR":
      return { ...state, step: "error", error: action.message };
    case "RESET":
      return initialState;
    default:
      return state;
  }
}
```

**Expected output:**
Reducer file created.

### Task 3: Multi-step checkout page

**What to do:**
Create `app/checkout/page.js` (this must be a Client Component to handle the interactive form):

```jsx
"use client";
import { useReducer } from "react";
import { checkoutReducer, initialState } from "./checkoutReducer";

export default function CheckoutPage() {
  const [state, dispatch] = useReducer(checkoutReducer, initialState);

  if (state.step === "cart") return <CartStep onNext={() => dispatch({ type: "NEXT" })} />;
  if (state.step === "info") return <InfoStep state={state} dispatch={dispatch} />;
  if (state.step === "payment") return <PaymentStep state={state} dispatch={dispatch} />;
  if (state.step === "processing") return <p>Processing...</p>;
  if (state.step === "confirmed") return <ConfirmedStep orderId={state.orderId} />;
  if (state.step === "error") return <p>Error: {state.error}</p>;
}
```

Build each step as its own small component. Info step collects customer data. Payment step picks a method (Cash on delivery / M-Pesa placeholder). Processing simulates a 1-second delay then dispatches SUCCESS with a fake order id.

**Expected output:**
You can walk through the whole flow. Back button works. Data preserved between steps.

### Task 4: Preserve data between steps

**What to do:**
Verify: fill in the info step, click next, click back, click next again. The info data should still be there. That is the point of the state machine.

**Expected output:**
Data preserved on back/forward navigation.

### Task 5: Notes on the design

**What to do:**
In `design-notes.md`, answer in 5-7 sentences:
- Why use a state machine instead of a bunch of booleans like `isInfoStep`, `isPaymentStep`?
- What happens if you refresh the page mid-checkout? How would you persist the state?
- What would you change if the payment took 30 seconds and you had to show progress?

Your own words.

**Expected output:**
`design-notes.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Persist checkout state to sessionStorage so a refresh keeps progress.
- Add form validation on the info step (email format, phone format).
- Add a stepper UI showing which step you are on.

## Submission Requirements

- **What to submit:** Repo with sketch, reducer, checkout page, `design-notes.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Paper sketch of checkout states | 15 | Committed. |
| Checkout reducer correct | 25 | All transitions work. |
| Multi-step page | 25 | You can walk from cart to confirmed. |
| Data preserved between steps | 15 | Back/forward does not lose info. |
| Design notes | 15 | Three questions answered in student's own words. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Letting each step manage its own state.** The state machine keeps everything in one place for a reason.
- **Using `useState` for `step`.** A reducer is better here because the transitions are constrained.
- **Forgetting the ERROR state.** Every real form can fail. Design for it.

## Resources

- Day 2 reading: [Checkout State Machine.md](./Checkout%20State%20Machine.md)
- Week 15 AI boundaries: [../ai.md](../ai.md)
