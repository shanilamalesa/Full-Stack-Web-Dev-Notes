# Week 15 - Day 1 Assignment

## Title
Cart State Machine -- Draw It On Paper First

## Overview
Week 15 builds the cart and checkout state machine. The habit this week: draw every state machine on paper before writing a line of code. Today you sketch the cart on paper, translate it into a `useReducer`, and persist it to localStorage.

## Learning Objectives Assessed
- Design a state machine on paper with states, transitions, and context
- Implement it with `useReducer`
- Persist state across reloads with localStorage
- Show cart count in a header component

## Prerequisites
- Week 14 completed

## AI Usage Rules

**Ratio this week:** 35% manual / 65% AI
**Habit:** Model the state machine on paper. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Reducer boilerplate after your paper sketch is finished.
- **NOT ALLOWED FOR:** Designing the state machine.
- **AUDIT REQUIRED:** Yes. Include a photo or ASCII sketch of your paper model.

## Tasks

### Task 1: Paper sketch

**What to do:**
On real paper (or a whiteboard), draw:
- **States:** `empty`, `populated`, `checking_out`
- **Transitions:** `add_item`, `remove_item`, `clear`, `proceed_to_checkout`, `back_to_cart`
- **Context:** the shape of a cart item (`{ productId, quantity, priceAtAdd }`)

Photograph it. Save as `assets/day1-sketch.jpg`.

**Expected output:**
Paper sketch committed.

### Task 2: useReducer implementation

**What to do:**
Create `app/cart/cartReducer.js`:

```javascript
export const initialState = { status: "empty", items: [] };

export function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM": {
      const { productId, priceCents, quantity = 1 } = action;
      const existing = state.items.find((i) => i.productId === productId);
      let items;
      if (existing) {
        items = state.items.map((i) =>
          i.productId === productId ? { ...i, quantity: i.quantity + quantity } : i
        );
      } else {
        items = [...state.items, { productId, priceCents, quantity }];
      }
      return { ...state, status: "populated", items };
    }
    case "REMOVE_ITEM": {
      const items = state.items.filter((i) => i.productId !== action.productId);
      return { ...state, items, status: items.length === 0 ? "empty" : "populated" };
    }
    case "CLEAR":
      return initialState;
    case "PROCEED":
      return { ...state, status: "checking_out" };
    case "BACK":
      return { ...state, status: "populated" };
    default:
      return state;
  }
}
```

**Expected output:**
Reducer file created and testable.

### Task 3: CartProvider with localStorage

**What to do:**
Create `app/cart/CartContext.jsx`:

```jsx
"use client";
import { createContext, useContext, useReducer, useEffect } from "react";
import { cartReducer, initialState } from "./cartReducer";

const CartContext = createContext(null);

export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState, (init) => {
    if (typeof window === "undefined") return init;
    const stored = localStorage.getItem("cart");
    return stored ? JSON.parse(stored) : init;
  });

  useEffect(() => {
    localStorage.setItem("cart", JSON.stringify(state));
  }, [state]);

  return <CartContext.Provider value={{ state, dispatch }}>{children}</CartContext.Provider>;
}

export function useCart() {
  const ctx = useContext(CartContext);
  if (!ctx) throw new Error("useCart must be inside CartProvider");
  return ctx;
}
```

Wrap your shop layout with `<CartProvider>`.

**Expected output:**
Cart persists across reloads.

### Task 4: "Add to cart" button

**What to do:**
On each product detail page, add a Client Component:

```jsx
"use client";
import { useCart } from "@/app/cart/CartContext";

export default function AddToCartButton({ productId, priceCents }) {
  const { dispatch } = useCart();
  return (
    <button onClick={() => dispatch({ type: "ADD_ITEM", productId, priceCents })}>
      Add to cart
    </button>
  );
}
```

Use it in the Server Component product page.

**Expected output:**
Clicking the button adds items to the cart and persists.

### Task 5: Cart count in header

**What to do:**
In your header (convert to Client Component or use a small Client wrapper), show the total number of items:

```jsx
"use client";
import { useCart } from "@/app/cart/CartContext";

export default function CartBadge() {
  const { state } = useCart();
  const count = state.items.reduce((sum, item) => sum + item.quantity, 0);
  return <span>Cart ({count})</span>;
}
```

**Expected output:**
Adding items updates the badge instantly.

## Stretch Goals (Optional - Extra Credit)

- Add a mini-cart dropdown showing the items.
- Add quantity controls (+ / -) on each cart item.
- Animate the cart badge on add.

## Submission Requirements

- **What to submit:** Repo with paper sketch photo, reducer, context, button, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Paper sketch committed | 15 | Real photo or ASCII sketch of the state machine. |
| useReducer implementation | 25 | All actions working. |
| CartProvider with localStorage | 20 | Cart persists across reloads. |
| Add to cart button on product page | 20 | Works from Server Component via Client wrapper. |
| Cart badge in header | 15 | Updates live. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Reading localStorage during SSR.** `typeof window === "undefined"` guard is required.
- **Forgetting the `"use client"` directive on components that use hooks.** Only Client Components can use `useReducer`.
- **Putting the provider above `<html>`.** Providers wrap `children`, not the whole layout root.

## Resources

- Day 1 reading: [The Cart State Machine.md](./The%20Cart%20State%20Machine.md)
- Week 15 AI boundaries: [../ai.md](../ai.md)
