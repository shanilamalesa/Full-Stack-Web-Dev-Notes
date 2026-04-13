# Week 15 - Day 4 Assignment

## Title
Order Management Admin Dashboard

## Overview
Day 4 is pre-weekend polish day. Today you build the admin side of the shop: list orders, view one order, update its status. This is a minimal but functional back-office. Week 16 adds payments on top; today the admin can at least see what customers bought.

## Learning Objectives Assessed
- List orders with pagination and filters
- View a single order with its items
- Update order status via a Server Action
- Protect admin routes with auth

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio:** 35/65. **Habit:** Paper first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Table UI and pagination boilerplate.
- **NOT ALLOWED FOR:** Auth protection or status update logic.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Admin orders list

**What to do:**
Create `app/admin/orders/page.js`:

```jsx
import pool from "@/lib/db";

export default async function AdminOrdersPage({ searchParams }) {
  const status = searchParams.status || "all";
  const limit = 20;
  const offset = parseInt(searchParams.offset || "0", 10);

  let sql = "SELECT id, customer_name, total_cents, status, created_at FROM orders";
  const params = [];
  if (status !== "all") {
    params.push(status);
    sql += ` WHERE status = $${params.length}`;
  }
  sql += ` ORDER BY created_at DESC LIMIT ${limit} OFFSET ${offset}`;

  const { rows } = await pool.query(sql, params);

  return (
    <main>
      <h1>Orders</h1>
      {/* filter links */}
      <table>
        <thead><tr><th>ID</th><th>Customer</th><th>Total</th><th>Status</th></tr></thead>
        <tbody>
          {rows.map((o) => (
            <tr key={o.id}>
              <td><a href={`/admin/orders/${o.id}`}>{o.id.slice(0, 8)}</a></td>
              <td>{o.customer_name}</td>
              <td>KES {(o.total_cents / 100).toLocaleString()}</td>
              <td>{o.status}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </main>
  );
}
```

**Expected output:**
Admin list shows real orders.

### Task 2: Status filter

**What to do:**
Add filter links at the top: `All`, `Pending`, `Paid`, `Fulfilled`, `Cancelled`. Each link sets `?status=...` in the URL. The Server Component reads `searchParams.status` and queries accordingly.

**Expected output:**
Clicking a filter narrows the list.

### Task 3: Order detail with status update

**What to do:**
Extend `app/admin/orders/[id]/page.js` (you built a similar page for customers on Day 3; copy/extend). Add a status dropdown powered by a Server Action:

```jsx
// app/admin/orders/[id]/updateStatus.js
"use server";
import pool from "@/lib/db";
import { revalidatePath } from "next/cache";

export async function updateStatus(orderId, newStatus) {
  await pool.query(
    "UPDATE orders SET status = $1 WHERE id = $2",
    [newStatus, orderId]
  );
  revalidatePath(`/admin/orders/${orderId}`);
  revalidatePath("/admin/orders");
}
```

In the page:

```jsx
// The form that calls the action
<form action={updateStatus.bind(null, order.id)}>
  <select name="newStatus" defaultValue={order.status}>
    <option value="pending">Pending</option>
    <option value="paid">Paid</option>
    <option value="fulfilled">Fulfilled</option>
    <option value="cancelled">Cancelled</option>
  </select>
  <button type="submit">Update</button>
</form>
```

Note: `.bind(null, order.id)` is the standard pattern for passing extra args to a Server Action.

**Expected output:**
Changing status persists and the list updates automatically via revalidatePath.

### Task 4: Admin auth

**What to do:**
Protect `/admin/*` routes with a simple auth check. Use your Week 12 JWT flow. Add middleware at `app/admin/layout.js`:

```jsx
import { cookies } from "next/headers";
import { redirect } from "next/navigation";
import jwt from "jsonwebtoken";

export default async function AdminLayout({ children }) {
  const token = cookies().get("auth_token")?.value;
  if (!token) redirect("/login");

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    if (payload.role !== "admin") redirect("/");
  } catch {
    redirect("/login");
  }

  return <>{children}</>;
}
```

You will need a login page that sets the `auth_token` cookie. Build a minimal one.

**Expected output:**
Visiting /admin/orders without a valid admin token redirects.

### Task 5: Pre-weekend checklist

**What to do:**
Create `CHECKLIST.md`:

```markdown
## Week 15 Day 4 Pre-Weekend Checklist

- [ ] Cart reducer with localStorage persistence
- [ ] Checkout state machine with 5 states
- [ ] Server Action creates orders in Postgres (transactional)
- [ ] Order detail page renders for a customer
- [ ] Admin orders list with filter
- [ ] Admin status update via Server Action
- [ ] Admin auth protects /admin/* routes
- [ ] Both paper sketches committed
- [ ] AI_AUDIT.md with sketches
```

Tick honestly.

**Expected output:**
Checklist committed.

## Stretch Goals (Optional - Extra Credit)

- Add a "quick view" modal for orders without leaving the list.
- Add a CSV export of filtered orders.
- Add an activity log (status change history) per order.

## Submission Requirements

- **What to submit:** Repo, admin pages, login, `CHECKLIST.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Admin orders list | 20 | Renders real orders. |
| Status filter | 10 | Query param drives filter. |
| Status update via Server Action | 25 | Persists and revalidates. |
| Admin auth protecting /admin | 25 | Unauthenticated users redirected. |
| Checklist complete | 10 | Honest. |
| Clean commits | 10 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Letting non-admins hit /admin pages.** Always check the role, not just "logged in".
- **Using `window.location.reload()` after status update.** `revalidatePath` is the Next.js way.
- **Forgetting to call `revalidatePath` in the Server Action.** The list will look stale until a hard refresh.

## Resources

- Day 4 reading: [Order Management and Admin.md](./Order%20Management%20and%20Admin.md)
- Week 15 AI boundaries: [../ai.md](../ai.md)
