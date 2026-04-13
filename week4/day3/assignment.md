# Week 4 - Day 3 Assignment

## Title
Parameters, Defaults, Rest, and Destructuring

## Overview
Today you met the three patterns that make JavaScript functions flexible: default parameters, rest parameters, and destructuring. Your assignment drills each one in isolation, then combines them in a small "receipt builder" function that accepts a variable number of line items.

## Learning Objectives Assessed
- Use default parameter values
- Use rest parameters (`...args`) to accept a variable number of arguments
- Destructure objects and arrays in function signatures
- Combine all three in a realistic function

## Prerequisites
- Day 1 and Day 2 completed

## AI Usage Rules

**Ratio:** 75/25. **Habit:** Build first, enhance with AI. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Extending the receipt builder with AI after your manual version works.
- **NOT ALLOWED FOR:** Generating the base implementation.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Default parameters drill

**What to do:**
In `js-lab/week4-day3/main.js`, write five functions with default parameters:

1. `greet(name = "friend")` returns "Hello, NAME!"
2. `raiseToPower(base, exponent = 2)` returns base^exponent
3. `formatPrice(amount, currency = "KSh")` returns "KSh 1,500"
4. `joinWords(words, separator = ", ")` joins words
5. `makeUser(name, { age = 18, city = "Nairobi" } = {})` returns a user object

Test each one with and without the optional parameters.

**Expected output:**
Five functions, each called twice (with and without defaults).

### Task 2: Rest parameters drill

**What to do:**
Write three functions that take variadic arguments:

1. `sumAll(...numbers)` sums any number of arguments
2. `concat(...strings)` joins any number of strings with a space
3. `maxOf(...values)` returns the largest of its arguments

Call each with different numbers of arguments (0, 1, 2, 5).

**Expected output:**
All three working with multiple call patterns.

### Task 3: Destructuring drill

**What to do:**
1. Given an array `[red, green, blue] = [255, 128, 64]`, destructure it in one line and use the values.
2. Given an object `{ name, age, city } = person`, destructure in a function parameter and log the fields.
3. Rename a destructured variable: `const { name: fullName } = user;`
4. Destructure with defaults: `const { theme = "light" } = preferences;`

**Expected output:**
Four working destructuring patterns.

### Task 4: Receipt builder (the combined challenge)

**What to do:**
Write a function `buildReceipt` that takes:
- A customer object `{ name, phone }`
- An optional tax rate (default 0.16)
- Any number of line items: each is an object `{ item, price, qty }`

It should return a formatted string like:

```
Receipt for Wanjiru (+254712000000)
--------------------------------
2 x Notebook @ KSh 350 = KSh 700
1 x Pen @ KSh 50 = KSh 50
--------------------------------
Subtotal: KSh 750
Tax (16%): KSh 120
Total: KSh 870
```

The signature should be:

```javascript
function buildReceipt({ name, phone }, taxRate = 0.16, ...items) {
  // ...
}
```

You must use destructuring on the first parameter, a default on the second, and rest on the third.

**Expected output:**
`buildReceipt` working with at least 3 test calls (different customers, different numbers of items).

### Task 5: Extend with AI

**What to do:**
After your `buildReceipt` works by hand, ask AI to extend it with a discount feature: a customer-level discount percentage that reduces the subtotal before tax. Record the full build-first log in `AI_AUDIT.md`.

**Expected output:**
Extended version and audit entry.

## Stretch Goals (Optional - Extra Credit)

- Add a second function `parseReceipt(text)` that takes a receipt string and returns an array of items. (Hard.)
- Support currency other than KSh via a parameter.
- Align the receipt columns using `padStart` or `padEnd` for a polished look.

## Submission Requirements

- **What to submit:** Repo with `js-lab/week4-day3/`, audit updated.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Default parameter drill | 15 | All 5 functions with defaults tested with and without. |
| Rest parameter drill | 15 | All 3 functions working with varying arg counts. |
| Destructuring drill | 15 | All 4 patterns demonstrated. |
| Receipt builder core | 25 | Destructuring, default, rest all used correctly in one signature. Output matches the format. Three test calls. |
| AI Audit extension | 15 | Build-first log complete. Discount feature added. |
| Clean commits | 10 | Conventional messages. |
| Code quality | 5 | No copy-paste from AI. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Rest parameter must be last.** `function f(...args, x)` is a syntax error. Rest always comes at the end of the parameter list.
- **Defaults only fill `undefined`, not `null`.** `greet(null)` will NOT use the default. It will use `null`. Test this yourself.
- **Destructuring without `= {}`.** `function f({ name }) {}` will crash if you call `f()` with no argument. Always add `= {}` as a safety default.

## Resources

- Day 3 reading: [Parameters, Rest & Default Values.md](./Parameters,%20Rest%20&%20Default%20Values.md)
- Week 4 AI boundaries: [../ai.md](../ai.md)
- MDN default parameters: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters
