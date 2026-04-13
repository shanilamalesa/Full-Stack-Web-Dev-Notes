# Week 16 - Day 2 Assignment

## Title
M-Pesa Callback Handler In Next.js

## Overview
Today you expose the M-Pesa callback endpoint inside your Next.js app and wire it to reconcile against the order's expected amount. This is the single most safety-critical route in your shop.

## Learning Objectives Assessed
- Expose a plain API route in Next.js (`app/api/.../route.js`)
- Verify the callback matches a known checkout request id
- Compare callback amount to expected amount
- Update the order status on success

## Prerequisites
- Day 1 completed

## AI Usage Rules

**Ratio:** 30/70. **Habit:** Money rule is permanent. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Route scaffolding only.
- **NOT ALLOWED FOR:** Amount comparison, idempotency, status update logic.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Create the callback route

**What to do:**
Create `app/api/mpesa/callback/route.js`:

```javascript
import pool from "@/lib/db";
import { NextResponse } from "next/server";

export async function POST(req) {
  const body = await req.json();
  const callback = body.Body?.stkCallback;
  if (!callback) return NextResponse.json({ status: "invalid" }, { status: 400 });

  const checkoutId = callback.CheckoutRequestID;

  // Look up order
  const { rows } = await pool.query(
    "SELECT id, total_cents, status FROM orders WHERE mpesa_checkout_id = $1",
    [checkoutId]
  );
  const order = rows[0];
  if (!order) {
    console.warn("Callback for unknown checkoutId:", checkoutId);
    return NextResponse.json({ status: "unknown" });
  }

  // Idempotency
  if (order.status === "paid") {
    console.log("Duplicate callback:", checkoutId);
    return NextResponse.json({ status: "already processed" });
  }

  const resultCode = callback.ResultCode;
  if (resultCode === 0) {
    const metadata = callback.CallbackMetadata?.Item || [];
    const amountReceived = metadata.find((i) => i.Name === "Amount")?.Value;
    const receipt = metadata.find((i) => i.Name === "MpesaReceiptNumber")?.Value;

    // Amount validation -- CRITICAL, hand-written
    const expectedCents = order.total_cents;
    const receivedCents = Math.round(Number(amountReceived) * 100);
    if (receivedCents !== expectedCents) {
      console.error(`AMOUNT MISMATCH: expected ${expectedCents}, got ${receivedCents}`);
      await pool.query(
        "UPDATE orders SET status = 'cancelled' WHERE id = $1",
        [order.id]
      );
      return NextResponse.json({ status: "mismatch" });
    }

    await pool.query(
      "UPDATE orders SET status = 'paid', mpesa_receipt = $1 WHERE id = $2",
      [receipt, order.id]
    );
  } else {
    await pool.query(
      "UPDATE orders SET status = 'cancelled' WHERE id = $1",
      [order.id]
    );
  }

  return NextResponse.json({ status: "ok" });
}
```

Add `mpesa_receipt TEXT` column to orders.

**Expected output:**
Route accepts POST and reconciles orders.

### Task 2: Update the callback URL in Daraja

**What to do:**
Your Daraja app points at a callback URL. Update it to your Next.js deployment URL (from Week 14 Day 5) + `/api/mpesa/callback`. Save in Daraja.

For local testing, use ngrok again to tunnel the Next.js dev server.

**Expected output:**
Daraja callback points to your new URL.

### Task 3: Test happy path

**What to do:**
Run the full flow: checkout, STK push, enter PIN in sandbox. Verify:
1. Callback arrives at your new route.
2. Order status updates to `paid`.
3. `mpesa_receipt` is populated.

**Expected output:**
Order status is `paid` after a successful payment. Screenshot `day2-paid.png`.

### Task 4: Test idempotency

**What to do:**
Re-POST the same callback body to your route (copy from server logs). The route should detect the duplicate and return early without re-processing.

**Expected output:**
Logs show "Duplicate callback". Order status unchanged.

### Task 5: Test amount mismatch

**What to do:**
Craft a fake callback POST with a wrong amount (use curl). Verify the route catches the mismatch and logs an error. The order status should become `cancelled`, not `paid`.

**Expected output:**
Mismatch detected. Screenshot `day2-mismatch.png`.

## Stretch Goals (Optional - Extra Credit)

- Add a status polling endpoint `/api/mpesa/status/:checkoutId` that the frontend can call.
- Add request logging so every callback is recorded for audit.
- Add a timeout: if no callback arrives within 90 seconds, mark the order `timed_out`.

## Submission Requirements

- **What to submit:** Repo, route, screenshots, `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Callback route created | 20 | POST handler working. |
| Amount validation (hand-written) | 30 | Mismatch caught and logged. |
| Idempotency | 20 | Duplicate callback is a no-op. |
| Happy path test | 15 | End-to-end works. |
| Money audit | 10 | Every money line flagged. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Trusting Daraja's amount.** Always compare to the stored intent. This is the most important rule in all of payments.
- **Returning 500 from the callback.** Daraja retries. Always return 200 and log errors for inspection.
- **Letting the frontend poll faster than every 3-5 seconds.** You will DOS your own API.

## Resources

- Day 2 reading: [Handling the M-Pesa Callback.md](./Handling%20the%20M-Pesa%20Callback.md)
- Week 16 AI boundaries: [../ai.md](../ai.md)
