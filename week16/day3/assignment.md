# Week 16 - Day 3 Assignment

## Title
WhatsApp Order Confirmations

## Overview
Today you wire up WhatsApp confirmations: when an order goes to `paid`, automatically send the customer a WhatsApp message confirming the order. You reuse your Week 11 WhatsApp bot and Meta Cloud API code.

## Learning Objectives Assessed
- Send outbound WhatsApp messages via Meta Cloud API
- Format a nice confirmation message
- Trigger the send from the M-Pesa callback handler
- Handle send failures without breaking the callback

## Prerequisites
- Days 1-2 completed
- Week 11 WhatsApp code still available

## AI Usage Rules

**Ratio:** 30/70. **Habit:** AI as integration engineer. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Message formatting, UI polish.
- **NOT ALLOWED FOR:** The send-on-paid trigger logic (that's your integration design).
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Send function

**What to do:**
Port your Week 11 `sendMessage` function into `lib/whatsapp.js` in the Next.js app:

```javascript
import axios from "axios";

export async function sendWhatsApp(to, text) {
  await axios.post(
    `https://graph.facebook.com/v18.0/${process.env.META_PHONE_NUMBER_ID}/messages`,
    {
      messaging_product: "whatsapp",
      to,
      text: { body: text },
    },
    { headers: { Authorization: `Bearer ${process.env.META_ACCESS_TOKEN}` } }
  );
}
```

Add env vars to Vercel and to local `.env.local`.

**Expected output:**
`lib/whatsapp.js` in place.

### Task 2: Confirmation message template

**What to do:**
Create a helper that builds the confirmation text:

```javascript
export function buildConfirmationMessage(order, items) {
  const lines = [
    `Asante! Your order is confirmed.`,
    `Order: ${order.id.slice(0, 8)}`,
    `Total: KES ${(order.total_cents / 100).toLocaleString()}`,
    `Items:`,
    ...items.map((i) => `- ${i.quantity}x ${i.name}`),
    `Track: https://YOUR_DOMAIN/orders/${order.id}`,
  ];
  return lines.join("\n");
}
```

**Expected output:**
Helper function returns the full message.

### Task 3: Trigger send from the callback

**What to do:**
Update your callback handler from Day 2. After marking an order as `paid`, send the WhatsApp confirmation:

```javascript
if (resultCode === 0 && receivedCents === expectedCents) {
  await pool.query(
    "UPDATE orders SET status = 'paid', mpesa_receipt = $1 WHERE id = $2",
    [receipt, order.id]
  );

  // Fetch customer phone and items for the message
  const { rows: orderRows } = await pool.query(
    "SELECT customer_phone FROM orders WHERE id = $1",
    [order.id]
  );
  const { rows: items } = await pool.query(
    "SELECT oi.quantity, p.name FROM order_items oi JOIN products p ON p.id = oi.product_id WHERE oi.order_id = $1",
    [order.id]
  );

  try {
    const text = buildConfirmationMessage(order, items);
    await sendWhatsApp(orderRows[0].customer_phone, text);
  } catch (err) {
    console.error("Failed to send WhatsApp confirmation:", err);
    // Do not fail the callback -- the order is paid, notification is best-effort
  }
}
```

**Expected output:**
After a successful payment, the customer receives a WhatsApp confirmation.

### Task 4: End-to-end test

**What to do:**
Run the full flow:
1. Add items to cart
2. Checkout with your real phone number (in international format)
3. Pay with M-Pesa sandbox
4. Receive the confirmation on WhatsApp

Screenshot the WhatsApp message as `day3-confirmation.png`.

**Expected output:**
Real confirmation arrives on your phone.

### Task 5: Failure handling

**What to do:**
Deliberately break the Meta access token (comment it out or corrupt it). Run another payment. Verify:
- The order still goes to `paid` (the payment succeeded)
- The callback handler logs the send failure but does not return an error

**Expected output:**
Graceful failure. Order is paid even if notification fails.

## Stretch Goals (Optional - Extra Credit)

- Use WhatsApp template messages for pre-approved messaging (real production pattern).
- Send a SMS fallback via Africa's Talking if WhatsApp send fails.
- Queue the send via BullMQ (preview of Week 23).

## Submission Requirements

- **What to submit:** Repo, `lib/whatsapp.js`, updated callback, screenshots, `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| WhatsApp send function | 15 | Works against sandbox. |
| Confirmation message template | 15 | Includes order details, items, tracking link. |
| Triggered from callback on payment success | 30 | Real WhatsApp arrives. |
| End-to-end test passes | 20 | Screenshot confirms. |
| Failure handling | 15 | Broken send does not break the order. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Awaiting the send inside a transaction.** Keep network calls out of DB transactions. If the network hangs, the DB holds the lock.
- **Letting the notification failure break the callback.** The payment succeeded -- the order must show as paid even if the message fails.
- **Hardcoding "hello world" instead of real order data.** Use the real order context.

## Resources

- Day 3 reading: [WhatsApp Order Confirmations.md](./WhatsApp%20Order%20Confirmations.md)
- Week 16 AI boundaries: [../ai.md](../ai.md)
- Week 11 WhatsApp assignments for the sender code.
