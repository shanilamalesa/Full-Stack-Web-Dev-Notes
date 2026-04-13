# Week 17 - Day 4 Assignment

## Title
Provider Selection And Fallbacks

## Overview
Day 4 is pre-weekend polish. Today you give customers a smart payment selection UI (based on their region and preferences), implement a fallback when the primary provider fails, and ship a final version of the multi-provider checkout.

## Learning Objectives Assessed
- Detect a hint about the user's region (header or locale)
- Suggest the best provider for that region
- Fall back to a secondary provider when the primary fails
- Track provider success rates

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio:** 30/70. **Habit:** Provider docs first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** UI for the selector, tables of providers.
- **NOT ALLOWED FOR:** The fallback logic (that is a design decision).
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Provider selector UI

**What to do:**
In the payment step, show three options grouped by region:

```
Kenya (recommended):
  (o) M-Pesa
  ( ) Airtel Money

International:
  ( ) Card via Stripe
```

Default to M-Pesa for +254 numbers, Stripe for everyone else.

**Expected output:**
Smart defaults based on the phone prefix.

### Task 2: Region-aware hint

**What to do:**
Read the customer's phone number. If it starts with `+254` or `254`, show a small "Recommended for Kenya" badge next to M-Pesa. Otherwise hide the badge and default to Stripe.

**Expected output:**
UI responds to phone number changes.

### Task 3: Fallback logic

**What to do:**
Design a fallback rule: if the primary provider errors out within 30 seconds, offer the user a "Try another payment method?" prompt.

Implement it in the AWAIT_PAYMENT state with a timer:

```jsx
useEffect(() => {
  const timer = setTimeout(() => {
    dispatch({ type: "OFFER_FALLBACK" });
  }, 30000);
  return () => clearTimeout(timer);
}, []);
```

When the user clicks "Try another", return to the PaymentStep with the failed provider disabled.

**Expected output:**
Fallback appears after 30 seconds of no update.

### Task 4: Track provider attempts

**What to do:**
Add a `payment_attempts` table:

```sql
CREATE TABLE payment_attempts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID REFERENCES orders(id),
  provider TEXT NOT NULL,
  status TEXT NOT NULL,
  error TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Log every attempt in your unified `initiatePayment` wrapper.

**Expected output:**
Every payment attempt (success or failure) logged.

### Task 5: Pre-weekend checklist

**What to do:**
Create `CHECKLIST.md`:

```markdown
## Week 17 Day 4 Pre-Weekend Checklist

- [ ] Stripe hosted checkout working
- [ ] Airtel collection working
- [ ] Unified initiatePayment interface in place
- [ ] Adapters return consistent shapes
- [ ] Provider selection UI with region hints
- [ ] Fallback after 30s of no response
- [ ] payment_attempts table logging every try
- [ ] Checkout UI has one code path
- [ ] AI_AUDIT.md with money audit current
```

Tick honestly.

**Expected output:**
Checklist committed.

## Stretch Goals (Optional - Extra Credit)

- Add provider success-rate tracking and auto-order the list accordingly.
- Add a "retry with same provider" option alongside fallback.
- Add admin page showing payment_attempts grouped by provider.

## Submission Requirements

- **What to submit:** Repo, tables, UI, `CHECKLIST.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Provider selector with defaults | 20 | Based on phone prefix. |
| Region hint badge | 10 | Shows for +254. |
| Fallback after 30s | 25 | Tested manually by blocking one provider. |
| payment_attempts logging | 25 | Every attempt recorded. |
| Checklist complete | 10 | Honest. |
| Clean commits | 10 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Re-running a payment without idempotency.** If the user clicks retry, make sure you do not double-charge on the original attempt. Use a new reference.
- **Showing Stripe to Kenyan users who would prefer M-Pesa.** Default matters more than availability.
- **Blocking the UI during fallback.** The user should be able to cancel at any time.

## Resources

- Day 4 reading: [Provider Selection and Fallbacks.md](./Provider%20Selection%20and%20Fallbacks.md)
- Week 17 AI boundaries: [../ai.md](../ai.md)
