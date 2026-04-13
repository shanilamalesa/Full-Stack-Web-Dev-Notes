# Week 15 - Day 3 Assignment

## Title
Server Actions And Form Mutations

## Overview
Today you replace the fake "processing" step with a real Server Action that writes an order to Postgres. Server Actions are Next.js's way to mutate data without building a REST API in between. They run on the server, can read secrets, and are called directly from Client Components via `<form action={...}>`.

## Learning Objectives Assessed
- Write a Server Action with `"use server"`
- Call a Server Action from a form
- Insert an order row in Postgres from a Server Action
- Use `redirect` after a successful mutation
- Handle errors returned from a Server Action

## Prerequisites
- Days 1-2 completed

## AI Usage Rules

**Ratio:** 35/65. **Habit:** Paper first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Server Action scaffolding.
- **NOT ALLOWED FOR:** Deciding what gets stored in the database.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Orders schema

**What to do:**
```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  customer_name TEXT NOT NULL,
  customer_email TEXT NOT NULL,
  customer_phone TEXT NOT NULL,
  customer_address TEXT NOT NULL,
  total_cents INTEGER NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'paid', 'fulfilled', 'cancelled')),
  payment_method TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE order_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id UUID NOT NULL REFERENCES products(id),
  quantity INTEGER NOT NULL,
  price_cents INTEGER NOT NULL
);
```

Load it into your hosted Postgres from Week 14.

**Expected output:**
Tables exist.

### Task 2: createOrder Server Action

**What to do:**
Create `app/checkout/actions.js`:

```javascript
"use server";

import pool from "@/lib/db";

export async function createOrder(formData) {
  const customerName = formData.get("name");
  const customerEmail = formData.get("email");
  const customerPhone = formData.get("phone");
  const customerAddress = formData.get("address");
  const paymentMethod = formData.get("paymentMethod");
  const itemsJson = formData.get("items");
  const items = JSON.parse(itemsJson);

  if (!customerName || !customerEmail || items.length === 0) {
    return { error: "Missing required fields" };
  }

  const total = items.reduce((sum, item) => sum + item.priceCents * item.quantity, 0);

  const client = await pool.connect();
  try {
    await client.query("BEGIN");
    const orderResult = await client.query(
      `INSERT INTO orders (customer_name, customer_email, customer_phone, customer_address, total_cents, payment_method)
       VALUES ($1, $2, $3, $4, $5, $6) RETURNING id`,
      [customerName, customerEmail, customerPhone, customerAddress, total, paymentMethod]
    );
    const orderId = orderResult.rows[0].id;

    for (const item of items) {
      await client.query(
        `INSERT INTO order_items (order_id, product_id, quantity, price_cents)
         VALUES ($1, $2, $3, $4)`,
        [orderId, item.productId, item.quantity, item.priceCents]
      );
    }

    await client.query("COMMIT");
    return { orderId };
  } catch (err) {
    await client.query("ROLLBACK");
    return { error: err.message };
  } finally {
    client.release();
  }
}
```

**Expected output:**
Server Action created. Both tables write in a transaction.

### Task 3: Wire the form

**What to do:**
In your checkout page's Payment step, replace the fake processing with a real form submission:

```jsx
"use client";
import { useTransition } from "react";
import { createOrder } from "./actions";
import { useCart } from "@/app/cart/CartContext";

export default function PaymentStep({ state, dispatch }) {
  const { state: cart, dispatch: cartDispatch } = useCart();
  const [isPending, startTransition] = useTransition();

  async function handleSubmit(formData) {
    formData.append("items", JSON.stringify(cart.items));
    const result = await createOrder(formData);
    if (result.error) {
      dispatch({ type: "ERROR", message: result.error });
    } else {
      cartDispatch({ type: "CLEAR" });
      dispatch({ type: "SUCCESS", orderId: result.orderId });
    }
  }

  return (
    <form action={(fd) => startTransition(() => handleSubmit(fd))}>
      <input type="hidden" name="name" value={state.customer.name} />
      <input type="hidden" name="email" value={state.customer.email} />
      <input type="hidden" name="phone" value={state.customer.phone} />
      <input type="hidden" name="address" value={state.customer.address} />
      <label>
        <input type="radio" name="paymentMethod" value="cash" required /> Cash on delivery
      </label>
      <label>
        <input type="radio" name="paymentMethod" value="mpesa" required /> M-Pesa (Week 16)
      </label>
      <button type="submit" disabled={isPending}>{isPending ? "Processing..." : "Place order"}</button>
    </form>
  );
}
```

**Expected output:**
Form submission calls the Server Action, order is created, cart is cleared, success state shown.

### Task 4: Order confirmation page

**What to do:**
Create `app/orders/[id]/page.js` (Server Component):

```jsx
import pool from "@/lib/db";
import { notFound } from "next/navigation";

export default async function OrderPage({ params }) {
  const { rows } = await pool.query("SELECT * FROM orders WHERE id = $1", [params.id]);
  const order = rows[0];
  if (!order) notFound();

  const { rows: items } = await pool.query(
    "SELECT oi.*, p.name FROM order_items oi JOIN products p ON p.id = oi.product_id WHERE oi.order_id = $1",
    [params.id]
  );

  return (
    <main>
      <h1>Order {order.id.slice(0, 8)}</h1>
      <p>Status: {order.status}</p>
      <p>Total: KES {(order.total_cents / 100).toLocaleString()}</p>
      <ul>
        {items.map((i) => (
          <li key={i.id}>{i.quantity}x {i.name}</li>
        ))}
      </ul>
    </main>
  );
}
```

From your checkout confirmed step, link to this page.

**Expected output:**
Confirmed step shows a link. Clicking it opens the order detail.

### Task 5: Transaction safety

**What to do:**
Test the failure path. Temporarily break the second query (e.g., insert into a non-existent table) and verify that the first query is rolled back. No partial writes should exist.

**Expected output:**
Rollback confirmed. No orphan orders in the DB.

## Stretch Goals (Optional - Extra Credit)

- Use `revalidatePath("/orders")` after creating an order so the list page refreshes.
- Add an optimistic UI update using `useOptimistic`.
- Return field-level errors instead of a single string.

## Submission Requirements

- **What to submit:** Repo with schema, action, wired form, order page, `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Orders schema | 15 | Both tables with correct constraints. |
| createOrder action | 30 | Writes both orders and order_items in a transaction. |
| Form wired to Server Action | 25 | Full flow works. Cart cleared on success. |
| Order detail page | 15 | Renders correctly. |
| Transaction rollback tested | 10 | No partial writes after deliberate failure. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Forgetting `"use server"` at the top of the actions file.** Without it, the function will ship to the client and leak secrets.
- **Using `fetch` from the client instead of a Server Action.** You could build a REST API, but the whole point this week is Server Actions.
- **Skipping the transaction.** A partial order is worse than no order.

## Resources

- Day 3 reading: [Server Actions and Form Mutations.md](./Server%20Actions%20and%20Form%20Mutations.md)
- Week 15 AI boundaries: [../ai.md](../ai.md)
- Server Actions docs: https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations
