# Week 16 - Day 4 Assignment

## Title
Notifications, Retries, Polish, And The Week 16 Ship Checklist

## Overview
Day 4 is pre-weekend polish day. Today you add a simple retry for failed WhatsApp sends, add a payment status UI that polls while the STK is pending, improve error messages, and finalise the Project 3 ship checklist.

## Learning Objectives Assessed
- Implement a basic retry with exponential backoff
- Poll an endpoint from the client for status updates
- Surface friendly error messages to the user
- Ship a complete Project 3 demo

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio:** 30/70. **Habit:** AI as integration engineer. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Retry boilerplate and UI polish.
- **NOT ALLOWED FOR:** Deciding the retry policy constants.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Retry helper for WhatsApp sends

**What to do:**
Create `lib/retry.js`:

```javascript
export async function withRetry(fn, { attempts = 3, baseDelayMs = 500 } = {}) {
  let lastErr;
  for (let i = 0; i < attempts; i++) {
    try {
      return await fn();
    } catch (err) {
      lastErr = err;
      if (i < attempts - 1) {
        await new Promise((r) => setTimeout(r, baseDelayMs * Math.pow(2, i)));
      }
    }
  }
  throw lastErr;
}
```

Use it around the WhatsApp send:

```javascript
await withRetry(() => sendWhatsApp(phone, text), { attempts: 3, baseDelayMs: 500 });
```

**Expected output:**
Retry wrapper in use. Failed sends retry twice before giving up.

### Task 2: Payment status polling

**What to do:**
Create `app/api/orders/[id]/status/route.js`:

```javascript
import pool from "@/lib/db";
import { NextResponse } from "next/server";

export async function GET(req, { params }) {
  const { rows } = await pool.query(
    "SELECT status FROM orders WHERE id = $1",
    [params.id]
  );
  if (!rows[0]) return NextResponse.json({ error: "Not found" }, { status: 404 });
  return NextResponse.json({ status: rows[0].status });
}
```

In your checkout AWAIT_PAYMENT state, poll this every 3 seconds:

```jsx
"use client";
import { useEffect } from "react";

export default function AwaitPayment({ orderId, onPaid, onFailed }) {
  useEffect(() => {
    const interval = setInterval(async () => {
      const res = await fetch(`/api/orders/${orderId}/status`);
      const data = await res.json();
      if (data.status === "paid") { clearInterval(interval); onPaid(); }
      if (data.status === "cancelled") { clearInterval(interval); onFailed(); }
    }, 3000);
    const timeout = setTimeout(() => clearInterval(interval), 120000); // 2 min cap
    return () => { clearInterval(interval); clearTimeout(timeout); };
  }, [orderId]);

  return <p>Check your phone for the M-Pesa prompt...</p>;
}
```

**Expected output:**
Polling updates the UI when the payment completes.

### Task 3: Friendly error messages

**What to do:**
Replace generic errors with user-friendly ones:

- "Payment initiation failed" -> "Could not reach M-Pesa. Please try again."
- "Mismatch" -> "Payment amount does not match the order. Please contact support."
- "Timed out" -> "No response from M-Pesa after 2 minutes. Check your phone or try again."

Display the error in your ERROR state step with a retry button.

**Expected output:**
Every error the customer can see is in plain language, not technical.

### Task 4: Ship checklist for Project 3

**What to do:**
Create `CHECKLIST.md`:

```markdown
## Week 16 Day 4 Project 3 Ship Checklist

- [ ] Next.js shop live on Vercel (or equivalent)
- [ ] Product catalogue with at least 8 real products
- [ ] Cart state machine with localStorage persistence
- [ ] Checkout flow: cart -> info -> payment -> processing -> confirmed
- [ ] M-Pesa STK push triggers from checkout
- [ ] M-Pesa callback reconciles orders with amount validation
- [ ] Idempotency prevents duplicate callback processing
- [ ] WhatsApp confirmation fires on successful payment
- [ ] Retry wraps WhatsApp send (best-effort)
- [ ] Payment status polling shows live updates in checkout
- [ ] Friendly error messages everywhere
- [ ] Admin /admin/orders shows all orders
- [ ] Admin can update status
- [ ] Admin routes protected by auth
- [ ] Money code audit up to date (zero AI-generated money lines)
- [ ] Repo pushed to main and deployed
```

Tick honestly. Any box that is not checked needs a one-line explanation.

**Expected output:**
`CHECKLIST.md` committed.

### Task 5: Demo video (optional for grading, required for submission if live demo is off)

**What to do:**
Record a 2-3 minute screen capture of you walking through the full shop flow: browse, add to cart, checkout, pay, receive WhatsApp, check order in admin. Upload somewhere accessible (Loom, Drive, YouTube unlisted).

**Expected output:**
Video link in your submission.

## Stretch Goals (Optional - Extra Credit)

- Add a "refund" button in the admin that calls a separate Server Action (stub it -- actual refunds come Week 18).
- Add a search box on the products page.
- Add estimated delivery time to the confirmation message.

## Submission Requirements

- **What to submit:** Live URL, repo, `CHECKLIST.md`, video (optional), `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Retry helper wrapping WhatsApp send | 15 | Works. |
| Status polling endpoint + UI | 25 | Live updates visible. |
| Friendly error messages | 10 | Plain language everywhere. |
| Ship checklist | 15 | Every box honestly ticked. |
| Full Project 3 working end-to-end | 25 | Live demo or video proves it. |
| Money audit current | 5 | Zero AI in money code. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Retrying everything.** Only retry transient errors (network). Do not retry a 400 Bad Request -- it will fail again.
- **Polling too fast.** 3-5 seconds is enough.
- **Showing stack traces to the user.** Log them to the server; show a friendly message.

## Resources

- Day 4 reading: [Notifications Retries and Polish.md](./Notifications%20Retries%20and%20Polish.md)
- Week 16 AI boundaries: [../ai.md](../ai.md)
