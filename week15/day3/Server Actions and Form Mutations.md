# Week 15, Day 3: Server Actions and Form Mutations

By the end of today, the "Pay" button on your checkout page calls a real Next.js Server Action that runs on the server, validates the input, inserts an `orders` row in Postgres, and returns the order id. No `/api` route, no `fetch` from the client, no JSON serialisation. The client calls the function; Next.js handles the RPC across the wire.

**Prior-week concepts you will use today:**
- Server Components and `"use client"` (Week 14, Day 1)
- The checkout state machine (Week 15, Day 2)
- Parameterised SQL and transactions (Week 12, Days 1-2)

**Estimated time:** 3-4 hours

---

## What Server Actions Are

A Server Action is a function you write in a file (or block) marked `"use server"`. You can import it into a Client Component and call it like any async function -- but when the client calls it, Next.js actually does an HTTP request in the background, runs the function on the server, and returns the result.

You write:

```javascript
// app/actions/createOrder.js
"use server";

export async function createOrder(formData) {
  // runs on the server
  const name = formData.get("name");
  const order = await db.query("INSERT INTO orders ...");
  return { orderId: order.id };
}
```

```jsx
// app/checkout/payment/PaymentForm.js
"use client";
import { createOrder } from "@/app/actions/createOrder";

async function handlePay() {
  const formData = new FormData();
  formData.append("name", info.name);
  const { orderId } = await createOrder(formData);
  // ...
}
```

Three things matter about this setup:

1. **No API route.** You did not write `app/api/orders/route.js`. Next.js generated the endpoint for you.
2. **No `fetch` or JSON.** The client calls the function directly. Next.js serialises `formData` over the wire and deserialises on the other side.
3. **The function has full server access.** It can read env vars, query the database, send emails, send SMS, and anything else a Node process can do.

This is Next.js's answer to the "how do I mutate data?" question. Server Components solve reads; Server Actions solve writes.

---

## The `orders` Schema

First, the database table. In `psql` against the `crm` database:

```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  customer_name TEXT NOT NULL,
  customer_phone TEXT NOT NULL,
  delivery_address TEXT NOT NULL,
  subtotal_cents INTEGER NOT NULL CHECK (subtotal_cents >= 0),
  payment_method TEXT NOT NULL CHECK (payment_method IN ('mpesa', 'airtel', 'cod')),
  status TEXT NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'paid', 'shipped', 'delivered', 'cancelled')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE order_items (
  id BIGSERIAL PRIMARY KEY,
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id UUID NOT NULL REFERENCES products(id),
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  price_cents_at_purchase INTEGER NOT NULL CHECK (price_cents_at_purchase >= 0)
);

CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_phone ON orders(customer_phone);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

Two decisions worth naming.

**`price_cents_at_purchase` on `order_items`.** When someone buys a phone for KSh 15,999 today and the price changes to KSh 17,999 tomorrow, their historical order should still say 15,999. Storing the price on the order item freezes it in time. The alternative -- joining on `products` to read the current price -- would silently rewrite history. This is one of the most common data modelling mistakes in beginner e-commerce apps.

**`REFERENCES products(id)` with no `ON DELETE` clause.** Default is `NO ACTION`. You cannot delete a product that has orders against it. This is correct: historical orders need to keep the product reference. If you want to hide a product from the catalogue, add an `archived` column and filter on it rather than deleting.

---

## The Server Action

Create `app/actions/createOrder.js`:

```javascript
// app/actions/createOrder.js
"use server";

import { pool } from "@/lib/db";
import { revalidatePath } from "next/cache";

function validatePhone(phone) {
  return /^\+?\d{9,14}$/.test(phone);
}

export async function createOrder(payload) {
  const { customer, items, paymentMethod } = payload;

  // Validation -- server side, never trust the client
  if (!customer?.name || customer.name.length < 2) {
    return { error: "Name is required" };
  }
  if (!validatePhone(customer.phone)) {
    return { error: "Phone number is invalid" };
  }
  if (!customer.address || customer.address.length < 5) {
    return { error: "Address is too short" };
  }
  if (!Array.isArray(items) || items.length === 0) {
    return { error: "Cart is empty" };
  }
  if (!["mpesa", "airtel", "cod"].includes(paymentMethod)) {
    return { error: "Invalid payment method" };
  }

  const client = await pool.connect();
  try {
    await client.query("BEGIN");

    // 1. Fetch current prices and stock for every product in one query
    const ids = items.map((i) => i.id);
    const { rows: products } = await client.query(
      `SELECT id, price_cents, stock FROM products WHERE id = ANY($1) FOR UPDATE`,
      [ids]
    );

    if (products.length !== items.length) {
      await client.query("ROLLBACK");
      return { error: "Some products were not found" };
    }

    const productMap = Object.fromEntries(products.map((p) => [p.id, p]));

    // 2. Check stock and compute subtotal from CURRENT server prices
    let subtotalCents = 0;
    for (const item of items) {
      const p = productMap[item.id];
      if (p.stock < item.quantity) {
        await client.query("ROLLBACK");
        return { error: `Out of stock: item ${item.id}` };
      }
      subtotalCents += p.price_cents * item.quantity;
    }

    // 3. Insert the order
    const { rows: orderRows } = await client.query(
      `INSERT INTO orders (customer_name, customer_phone, delivery_address, subtotal_cents, payment_method, status)
       VALUES ($1, $2, $3, $4, $5, $6)
       RETURNING id`,
      [
        customer.name,
        customer.phone,
        customer.address,
        subtotalCents,
        paymentMethod,
        paymentMethod === "cod" ? "pending" : "pending", // real payment moves to 'paid' via webhook in wk16
      ]
    );
    const orderId = orderRows[0].id;

    // 4. Insert order items
    for (const item of items) {
      const p = productMap[item.id];
      await client.query(
        `INSERT INTO order_items (order_id, product_id, quantity, price_cents_at_purchase)
         VALUES ($1, $2, $3, $4)`,
        [orderId, item.id, item.quantity, p.price_cents]
      );
    }

    // 5. Deduct stock
    for (const item of items) {
      await client.query(
        `UPDATE products SET stock = stock - $1 WHERE id = $2`,
        [item.quantity, item.id]
      );
    }

    await client.query("COMMIT");

    // 6. Invalidate the catalogue cache so stock counts refresh
    revalidatePath("/products");
    revalidatePath(`/products/[slug]`, "page");

    return { orderId };
  } catch (err) {
    await client.query("ROLLBACK");
    console.error("createOrder failed:", err);
    return { error: "Order failed. Please try again." };
  } finally {
    client.release();
  }
}
```

This is the most important function in the shop. Read every line. Six things are happening:

**1. Validation first.** The client may be compromised, edited, or just buggy. The server re-validates every field. If anything is wrong, we return an error object (not throw) so the client can show it nicely.

**2. Get a dedicated client for the transaction.** We can only `BEGIN`/`COMMIT`/`ROLLBACK` on a single connection, not on the pool's `query` shortcut. `pool.connect()` borrows one client for the whole transaction.

**3. Recompute prices on the server.** We never trust the `price_cents` from the cart. The client might have been sitting on the checkout page for an hour; prices may have changed. The order total is always the sum of current server prices at the moment of purchase.

**4. `FOR UPDATE` on the product SELECT.** This locks the rows so a parallel order cannot oversell the last unit. Without it, two simultaneous checkouts could both see `stock = 1` and both succeed. With it, one transaction waits for the other.

**5. Stock deduction is part of the same transaction.** Either all of inserting the order, inserting items, and deducting stock happens, or none of it does. A crash between step 4 and step 5 would leave an order with no stock deduction; the transaction prevents that.

**6. `revalidatePath`** invalidates the Next.js cache. The `/products` page will re-query the database on the next visitor so the new stock count is visible.

---

## Calling The Action From The Client

Update `app/checkout/payment/PaymentForm.js` -- replace the fake `handlePay` with a real one:

```jsx
import { createOrder } from "@/app/actions/createOrder";

async function handlePay() {
  setSubmitting(true);
  setErrorMessage("");
  setPaymentMethod(method);

  const result = await createOrder({
    customer: info,
    items: items.map((i) => ({ id: i.id, quantity: i.quantity })),
    paymentMethod: method,
  });

  if (result.error) {
    setErrorMessage(result.error);
    setSubmitting(false);
    return;
  }

  setOrderId(result.orderId);
  clearCart();
  goTo(STATES.CONFIRMED);
  router.push("/checkout/confirmed");
}
```

Add local state for the error message:

```jsx
const [errorMessage, setErrorMessage] = useState("");
// ...render near the button:
{errorMessage && <p className="text-red-600 text-sm">{errorMessage}</p>}
```

Now try it:
1. Fill in the form, click Pay.
2. Check Postgres -- there should be a new `orders` row with all your info, a `subtotal_cents` matching what you saw, and corresponding `order_items`.
3. Go back to `/products` -- the stock counts have dropped by the amounts you bought.
4. Clear the cart, add 50 of a product with stock 5, and checkout -- you should see "Out of stock". The order did not get created.

### Artificial race test

Open two browser tabs. In both, add the last unit of a product with stock 1 to the cart. Fill in checkout info in both. Click Pay at the *same time* in both tabs. One should succeed; the other should show "Out of stock". If both succeed, your `FOR UPDATE` is not working -- check that it is on the SELECT, not on the UPDATE.

This is the hardest test to reproduce manually and the most important one to get right. Real Friday evening traffic on a boda service will surface races like this; an hour now saves you an incident later.

---

## Progressive Enhancement With `<form action>`

Server Actions have a second form that works without any JavaScript on the client at all. Instead of calling the function from `onClick`, you pass the function to `<form action={...}>`:

```jsx
<form action={createOrder}>
  <input name="name" />
  <input name="phone" />
  <button type="submit">Submit</button>
</form>
```

When submitted, Next.js intercepts the form, runs the action on the server, and handles the result. If JavaScript is disabled on the client, the form still works -- it submits through a traditional POST, the server runs the action, and the page re-renders. This is the "progressive enhancement" pattern Next.js is particularly good at.

For checkout, we use the function form because we want access to local state and client-side cart. For simpler forms (contact, search, admin edits) use the `action` prop and get JS-less support free.

---

## Optimistic Updates With `useOptimistic`

One more tool you should know. When a user changes cart quantity, the network call to the server takes some milliseconds. During that window, the UI should either show a loading state or pretend the change already happened. The latter is an "optimistic update" -- nicer UX but risky if the server disagrees.

React 19 ships a `useOptimistic` hook that formalises this. Example usage in a cart:

```jsx
"use client";
import { useOptimistic } from "react";
import { incrementItem } from "@/app/actions/cartActions";

export default function CartRow({ item }) {
  const [optimisticQty, setOptimisticQty] = useOptimistic(item.quantity);

  async function handleIncrement() {
    setOptimisticQty(optimisticQty + 1);
    await incrementItem(item.id);
  }

  return (
    <div>
      <span>{optimisticQty}</span>
      <button onClick={handleIncrement}>+</button>
    </div>
  );
}
```

For today's checkout, we do not need it -- the Pay click is a one-shot action and the user is fine seeing a "Processing..." state for two seconds. But keep `useOptimistic` in mind for the admin pages in Week 16.

---

## Checkpoint

1. Filling out the checkout flow inserts a real `orders` row and linked `order_items` rows.
2. The total on the order matches the `items.quantity * products.price_cents` sum computed on the server.
3. Stock on purchased products decreases by the bought amount.
4. An order for an out-of-stock product fails cleanly with a visible error.
5. Two simultaneous checkouts for the last unit result in one success and one failure.
6. `psql` shows the order as `status = 'pending'` (real payment confirmation is Week 16).
7. Reloading `/products` after an order shows updated stock counts.
8. The server action never trusts the client's `price_cents` -- the test for this is setting `item.price_cents` to 1 in devtools before clicking Pay; the server still charges the real price.

Commit:

```bash
git add .
git commit -m "feat: createOrder server action with transactional stock deduction"
```

---

## What You Learned

- Server Actions replace API routes for mutation.
- `"use server"` marks a file or block as server-only code callable from the client.
- Transactions with a borrowed pool client ensure atomicity on multi-step writes.
- `FOR UPDATE` locks rows to prevent oversell.
- Never trust client-supplied prices; recompute on the server.
- `revalidatePath` clears Next.js caches so other visitors see new data.

Tomorrow we tighten the order flow: customer account linking, a "my orders" page, inventory restock, order status transitions on the admin side. Friday is the recap and the weekend is end-to-end polish.
