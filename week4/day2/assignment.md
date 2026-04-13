# Week 4 - Day 2 Assignment

## Title
Arrow Functions, Implicit Returns, and Functional Array Methods

## Overview
Today you learned the arrow function syntax and its subtle `this` difference. Your assignment drills the shape until your fingers know it, then applies arrow functions to the three most-used array methods -- `map`, `filter`, and `reduce` -- which will appear in every week from now on.

## Learning Objectives Assessed
- Convert function expressions into arrow functions
- Use implicit returns correctly
- Apply `.map`, `.filter`, `.reduce` to arrays with arrow callbacks
- Recognise when NOT to use an arrow (method definitions on objects)

## Prerequisites
- Day 1 completed

## AI Usage Rules

**Ratio:** 75/25. **Habit:** Build first, enhance with AI. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Extending a working `reduce` with AI after you finished your own version.
- **NOT ALLOWED FOR:** Writing the first `map`, `filter`, or `reduce` call for you.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Convert declarations to arrows

**What to do:**
Take all 10 functions from Day 1 Task 1 and rewrite each one as an arrow function. Save them in a new file `js-lab/week4-day2/arrows.js`. Use implicit returns where the body is a single expression.

Example:

```javascript
// Before
function square(n) { return n * n; }

// After
const square = (n) => n * n;
```

**Expected output:**
`arrows.js` with 10 arrow function versions, each working with the same test cases as Day 1.

### Task 2: map drill

**What to do:**
Given this array:

```javascript
const products = [
  { name: "Notebook", price_cents: 35000 },
  { name: "Pen", price_cents: 5000 },
  { name: "Laptop Stand", price_cents: 120000 },
];
```

Use `.map` to produce:
1. An array of just the names
2. An array of just the prices in shillings (price_cents / 100)
3. An array of new objects with an added `priceFormatted` property like `"KSh 350"`

**Expected output:**
Three separate map results logged to the console.

### Task 3: filter drill

**What to do:**
Using the same `products` array, use `.filter` to produce:
1. Products that cost less than KSh 1000 (price_cents < 100000)
2. Products whose name has more than 5 characters

**Expected output:**
Two filter results logged.

### Task 4: reduce drill

**What to do:**
Use `.reduce` to compute:
1. The total price of all products in cents
2. A single object mapping product name to price: `{ Notebook: 35000, Pen: 5000, ... }`

**Expected output:**
Two reduce results logged.

### Task 5: When NOT to use arrow

**What to do:**
In a short file `arrow-notes.md`, explain with one code example why you would NOT use an arrow for a method defined on an object. Use the `this` binding difference as the reason.

Example of the issue:

```javascript
const counter = {
  count: 0,
  // arrow `this` does NOT point at counter
  increment: () => { this.count += 1; },
};
counter.increment();
console.log(counter.count); // still 0, not 1
```

Explain in 3-4 sentences why. No AI for the explanation.

**Expected output:**
`arrow-notes.md` with the example and explanation.

## Stretch Goals (Optional - Extra Credit)

- Chain `.filter(...).map(...).reduce(...)` into a single statement that returns the total cost of products under KSh 1000.
- Use `Array.from` with a mapper to generate an array of squares from 1 to 10 in one line.
- Research `Array.prototype.flatMap` and write one example using it.

## Submission Requirements

- **What to submit:** Repo with `js-lab/week4-day2/`, `arrow-notes.md`, updated `AI_AUDIT.md`.
- **Where:** Week 4 submissions channel
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| 10 arrow conversions | 20 | All 10 Day 1 functions converted to arrows. Implicit returns used where appropriate. |
| map drill (3 results) | 15 | All three map outputs correct. |
| filter drill (2 results) | 15 | Both filter outputs correct. |
| reduce drill (2 results) | 20 | Sum and object-accumulator both working. The object-accumulator is the harder one -- points awarded only if fully correct. |
| Arrow-vs-method notes | 15 | `this` difference explained with code example. |
| AI Audit | 10 | Build-first log entry updated. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Implicit return with a body block.** `() => { value }` does NOT return value. It has a block body so you need an explicit return. Implicit return only works with a single expression (no braces): `() => value`.
- **Forgetting the initial value in reduce.** Always pass the initial value as the second argument: `arr.reduce(fn, initialValue)`. Omitting it is a common bug source.
- **Using arrow functions for object methods when you need `this`.** The whole point of arrows is they don't rebind `this`. That is great for callbacks, bad for methods that need to reference their own object.

## Resources

- Day 2 reading: [Arrow Functions & Implicit Returns.md](./Arrow%20Functions%20&%20Implicit%20Returns.md)
- Week 4 AI boundaries: [../ai.md](../ai.md)
- MDN Array methods: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
