# Week 15, Day 1: The Cart State Machine

By the end of today, your shop has a working cart. Users can add items, remove items, change quantities, see a cart count in the header, and the cart persists across page reloads and tabs. You will build it with Zustand for client-side state and `localStorage` for persistence. The cart lives entirely on the client today; tomorrow's checkout is where the server gets involved.

**Prior-week concepts you will use today:**
- Server Components vs Client Components (Week 14, Day 1)
- The state machine pattern (Week 13, Day 2 -- USSD states)
- The Next.js product catalogue (Week 14 full)

**Estimated time:** 3 hours

---

## The Week Ahead

| Day | What You Build |
|---|---|
| Day 1 (today) | Client-side cart with Zustand. Add, remove, change quantity, persist to localStorage, header counter. |
| Day 2 | Checkout state machine -- the states (browsing/cart/info/payment/confirmed) and the transitions. |
| Day 3 | Server Actions for form submission, validation, and the first real order write to Postgres. |
| Day 4 | Orders schema, inventory deduction inside a transaction, order confirmation page. |
| Day 5 | Week recap + peer coding. |
| Weekend | A working shop you can run end to end -- browse, add to cart, checkout with a fake "Pay" button. Real payments are Week 16. |

---

## Why A State Machine, Again

You already wrote state machines for the USSD app in Week 13. The cart is the same pattern, applied to the web. Both have:

- A small set of **states** (`empty`, `has_items`, `checking_out`, `payment_pending`, `confirmed`).
- **Transitions** triggered by user actions (`add`, `remove`, `checkout`, `pay_ok`, `pay_fail`).
- **Context** that the transitions read and write (items, totals, customer info).

Why use it here instead of a tangle of `useState` calls? Because cart logic gets complicated fast:

- Adding a product that is already in the cart should increment quantity, not duplicate.
- Setting a quantity to 0 should remove the item.
- The cart total must always reflect the current state.
- The cart must survive navigation, reloads, and new tabs.
- The UI must stay in sync with all of the above across multiple components (header, cart page, checkout).

Three `useState` hooks become six in a week, then get tangled with `useEffect`, and the bugs arrive. A single store with well-defined actions is clean, testable, and easy to reason about.

---

## Installing Zustand

Zustand is a tiny state management library -- 1.2 KB, no provider needed, and API you can learn in ten minutes.

```bash
cd shop
npm install zustand
```

That is the whole install. No boilerplate, no context wrapper in the root layout. Let us build the store.

---

## The Cart Store

Create `app/lib/cartStore.js`:

```javascript
// app/lib/cartStore.js
"use client";

import { create } from "zustand";
import { persist } from "zustand/middleware";

const initialState = {
  items: [],
};

export const useCart = create(
  persist(
    (set, get) => ({
      ...initialState,

      addItem(product, quantity = 1) {
        const items = get().items;
        const existing = items.find((i) => i.id === product.id);

        if (existing) {
          set({
            items: items.map((i) =>
              i.id === product.id ? { ...i, quantity: i.quantity + quantity } : i
            ),
          });
        } else {
          set({
            items: [
              ...items,
              {
                id: product.id,
                slug: product.slug,
                name: product.name,
                price_cents: product.price_cents,
                image_url: product.image_url,
                quantity,
              },
            ],
          });
        }
      },

      removeItem(productId) {
        set({ items: get().items.filter((i) => i.id !== productId) });
      },

      setQuantity(productId, quantity) {
        if (quantity <= 0) {
          get().removeItem(productId);
          return;
        }
        set({
          items: get().items.map((i) =>
            i.id === productId ? { ...i, quantity } : i
          ),
        });
      },

      clear() {
        set(initialState);
      },

      // Derived state -- computed on every read.
      totalItems() {
        return get().items.reduce((sum, i) => sum + i.quantity, 0);
      },

      totalCents() {
        return get().items.reduce((sum, i) => sum + i.quantity * i.price_cents, 0);
      },
    }),
    {
      name: "mctaba-cart",
      // Only persist items, not the functions
      partialize: (state) => ({ items: state.items }),
    }
  )
);
```

There is a lot going on -- read it slowly.

**`"use client"`** -- this file uses client-only APIs (`localStorage` through `persist`), so Zustand stores are always Client Components.

**`create(...)`** returns a hook. Calling `useCart()` in a component gives you the current state and every action. Components that read from the store automatically re-render when the data changes.

**`set` and `get`** -- inside the store, `set` replaces the state and `get` reads the latest state. Crucial: always call `get().items` inside an action, not the stale `items` from the closure. Otherwise you introduce subtle bugs when two actions fire back to back.

**`persist` middleware** -- wraps the store in `localStorage`. Every `set` is serialised to `localStorage` under the key `mctaba-cart`. Refreshing the page or opening a new tab restores the cart automatically. The `partialize` option is important: without it, Zustand would try to serialise the action functions to localStorage, which does not work and throws on reload.

**Derived state (`totalItems`, `totalCents`)** -- these are functions, not stored values. They compute from `items` on every read. Keeping them as functions means they are always in sync; storing them as values would require recomputing on every change, and it is easy to forget.

### Alternative patterns

You can use plain `useState` in a shared context provider, or `useReducer`, or Redux Toolkit. Zustand wins for small-to-medium apps because:

- No provider component to wrap the root in.
- No boilerplate for actions and reducers.
- Works with React Server Components (no provider means no forced "use client" on the whole tree).
- `persist` is built in.

For an e-commerce cart this is ideal. For a large app with hundreds of interacting pieces of state, you would consider Redux Toolkit or Jotai. Pick the right tool for the size.

---

## Wiring The Header Counter

In `app/layout.js`, the header is currently a Server Component. We want to show a "Cart (3)" indicator that updates live as items are added. That is client-side state, so the counter itself must be a Client Component.

Create `app/components/CartIndicator.js`:

```jsx
// app/components/CartIndicator.js
"use client";

import Link from "next/link";
import { useCart } from "@/app/lib/cartStore";
import { useEffect, useState } from "react";

export default function CartIndicator() {
  const totalItems = useCart((s) => s.totalItems());
  const [hydrated, setHydrated] = useState(false);

  useEffect(() => setHydrated(true), []);

  return (
    <Link href="/cart" className="relative">
      Cart
      {hydrated && totalItems > 0 && (
        <span className="ml-2 bg-brand text-white text-xs rounded-full px-2 py-0.5">
          {totalItems}
        </span>
      )}
    </Link>
  );
}
```

Two things are worth naming.

**`useCart((s) => s.totalItems())`** -- the selector form. You pass a function that picks the piece of state you want, and Zustand only re-renders this component when *that* piece changes. If you instead did `const cart = useCart()` and then `cart.totalItems()`, the component would re-render on every cart change even when the count did not change. Selectors are the performance discipline of Zustand.

**The `hydrated` flag** -- on first render, the server has no `localStorage`, so it renders with 0 items. Then the client hydrates and the real count appears. Without the flag, React complains about the mismatch. Using `hydrated` ensures the counter is rendered only after client takeover. This is a standard Next.js + persistent-store pattern; memorise it.

Now use the component in `app/layout.js`:

```jsx
import CartIndicator from "./components/CartIndicator";

// ...inside the nav:
<div className="flex gap-6">
  <Link href="/products/category/phones">Phones</Link>
  <Link href="/about">About</Link>
  <Link href="/contact">Contact</Link>
  <CartIndicator />
</div>
```

The root layout is still a Server Component. The `<CartIndicator />` is a Client Component imported into it. Next.js renders the layout on the server, and for the `CartIndicator` it sends a placeholder plus the JS bundle for the component. The client takes over the placeholder on hydration. Same pattern as the `Counter` from Week 14 Day 1.

---

## Wiring The "Add To Cart" Button

Update `app/components/AddToCartButton.js`:

```jsx
// app/components/AddToCartButton.js
"use client";
import { useCart } from "@/app/lib/cartStore";
import { useState } from "react";
import Button from "./Button";

export default function AddToCartButton({ product }) {
  const addItem = useCart((s) => s.addItem);
  const [added, setAdded] = useState(false);

  function handleClick() {
    addItem(product, 1);
    setAdded(true);
    setTimeout(() => setAdded(false), 1500);
  }

  return (
    <Button disabled={product.stock === 0} onClick={handleClick}>
      {product.stock === 0 ? "Out of stock" : added ? "Added!" : "Add to cart"}
    </Button>
  );
}
```

Click "Add to cart" on any product page. Check the header -- the counter goes to 1. Click again -- 2. Reload the page -- still 2. Open a new tab at `localhost:3000` -- still 2, because `localStorage` is shared.

This is the payoff of `persist`. One line of middleware, and the cart survives reloads, refreshes, and tabs for free.

---

## The Cart Page

Create `app/cart/page.js`:

```jsx
// app/cart/page.js
import CartView from "./CartView";

export const metadata = { title: "Your Cart" };

export default function CartPage() {
  return (
    <div className="max-w-3xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-6">Your Cart</h1>
      <CartView />
    </div>
  );
}
```

Notice: `page.js` is still a Server Component. The interactive part is `CartView`, which is a Client Component. The server renders the page shell with the heading; the client hydrates the cart details.

Create `app/cart/CartView.js`:

```jsx
// app/cart/CartView.js
"use client";

import Link from "next/link";
import { useCart } from "@/app/lib/cartStore";
import { useEffect, useState } from "react";

export default function CartView() {
  const items = useCart((s) => s.items);
  const totalCents = useCart((s) => s.totalCents());
  const setQuantity = useCart((s) => s.setQuantity);
  const removeItem = useCart((s) => s.removeItem);
  const [hydrated, setHydrated] = useState(false);

  useEffect(() => setHydrated(true), []);

  if (!hydrated) return <p>Loading...</p>;

  if (items.length === 0) {
    return (
      <div className="text-center py-12">
        <p className="text-gray-500 mb-4">Your cart is empty.</p>
        <Link href="/products" className="bg-brand text-white px-6 py-3 rounded">
          Browse products
        </Link>
      </div>
    );
  }

  return (
    <div className="space-y-4">
      {items.map((item) => (
        <div key={item.id} className="flex items-center justify-between border-b pb-4">
          <div>
            <Link href={`/products/${item.slug}`} className="font-medium">
              {item.name}
            </Link>
            <p className="text-sm text-gray-600">
              KSh {(item.price_cents / 100).toLocaleString()} each
            </p>
          </div>
          <div className="flex items-center gap-3">
            <input
              type="number"
              min="1"
              value={item.quantity}
              onChange={(e) => setQuantity(item.id, parseInt(e.target.value, 10) || 0)}
              className="w-16 border p-1 text-center"
            />
            <button
              onClick={() => removeItem(item.id)}
              className="text-red-600 text-sm"
            >
              Remove
            </button>
          </div>
        </div>
      ))}

      <div className="flex items-center justify-between pt-4 text-xl font-bold">
        <span>Total</span>
        <span>KSh {(totalCents / 100).toLocaleString()}</span>
      </div>

      <Link
        href="/checkout"
        className="block text-center bg-brand text-white py-3 rounded mt-6"
      >
        Proceed to checkout
      </Link>
    </div>
  );
}
```

Tested behaviours:

1. Empty cart shows the empty state with a CTA.
2. Quantity input updates the count live; total updates; header counter updates.
3. Setting quantity to 0 or clicking "Remove" removes the item.
4. Reload -- cart still there.

---

## A Note On Hydration

The `useEffect(() => setHydrated(true), [])` trick appears twice in today's code. It is the canonical solution to the "server rendered a different tree" warning when your state depends on `localStorage`. The flow is:

1. Server renders. `items = []` because server has no localStorage. HTML sent.
2. Client receives HTML, React attaches.
3. Zustand reads localStorage, now `items = [3 products]`.
4. React re-renders.
5. Without the flag, React compared step 1's HTML to step 3's first render and yelled about mismatch.
6. With the flag, the component returns the empty/loading state until `hydrated === true`, matching the server render.

It is ugly. It is the right ugly. Next.js 15 is moving toward a better primitive (`useSyncExternalStore`) but for now, the flag is standard.

---

## Checkpoint

1. Adding a product updates the header cart counter.
2. Reloading keeps the cart intact.
3. Opening a new tab shows the same cart.
4. The cart page lists items, lets you change quantity, and shows the total.
5. Setting quantity to 0 removes the item.
6. `Clear` (there is no button yet -- add one if you want) empties the cart.
7. `localStorage.getItem("mctaba-cart")` in the devtools console shows the serialised state.

Commit:

```bash
git add .
git commit -m "feat: client-side cart with zustand and localStorage persistence"
```

---

## What You Learned

- Zustand gives you a tiny, provider-free state store with persistence middleware.
- Selectors prevent unnecessary re-renders.
- The `hydrated` flag is the standard escape hatch for client-only state on SSR pages.
- The cart page is a Server Component wrapping a Client Component -- the standard Next.js pattern for "interactive page with server-rendered shell".

Tomorrow we start the checkout. That is where Server Actions enter the story and the cart starts talking to the backend.
