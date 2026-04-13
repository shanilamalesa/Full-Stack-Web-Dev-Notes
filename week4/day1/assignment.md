# Week 4 - Day 1 Assignment

## Title
Functions and First DOM Manipulation

## Overview
Today you met two of the biggest tools in JavaScript: function declarations / expressions, and the DOM. Your assignment is to write 10 small functions by hand, then use them to manipulate a real HTML page through `document.querySelector` and event listeners. By the end of this assignment you will have a small interactive page where clicking buttons changes things on screen.

## Learning Objectives Assessed
- Write function declarations and function expressions and explain when each is appropriate
- Use parameters and return values correctly
- Select DOM elements with `getElementById`, `querySelector`, and `querySelectorAll`
- Modify text content, classes, and styles via JavaScript
- Attach `click` event listeners to DOM elements

## Prerequisites
- Completed Week 3 assignments
- Day 1 readings: [Function Declarations & Expressions.md](./Function%20Declarations%20&%20Expressions.md) and [JavaScript-DOM.md](./JavaScript-DOM.md)

## AI Usage Rules

**Ratio this week:** 75% manual / 25% AI
**Habit:** Build first, enhance with AI -- the manual version must work before AI touches it. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Asking AI to extend a working function with a new feature you can describe in one sentence. Asking AI to explain a DOM API after you tried it.
- **NOT ALLOWED FOR:** Generating your first 10 functions. Generating event handler code from scratch.
- **AUDIT REQUIRED:** Yes. Include a "Build-first log" section showing pseudocode -> manual version -> AI extension for at least one function.

## Tasks

### Task 1: Write 10 functions

**What to do:**
In `js-lab/week4-day1/main.js`, write these 10 functions by hand. No AI for the first version.

1. `square(n)` returns n squared
2. `greet(name)` returns "Hello, NAME!"
3. `isEven(n)` returns true if n is even
4. `max(a, b)` returns the larger of two numbers
5. `sum(arr)` takes an array of numbers and returns their sum
6. `firstWord(sentence)` returns the first word of a string
7. `reverseString(s)` returns the string reversed
8. `countVowels(s)` returns the count of vowels in a string
9. `range(start, end)` returns an array of integers from start to end inclusive
10. `clamp(value, min, max)` returns value bounded by min and max

Test each one with at least two inputs and `console.log` the result.

**Expected output:**
All 10 functions defined, all 20+ test calls visible in console output.

### Task 2: Function declaration vs expression

**What to do:**
Pick three of the functions above and rewrite each one as both a function declaration and a function expression assigned to a `const`. In a comment for each pair, note one practical difference (hoisting, naming, what `this` binds to).

Example:

```javascript
// declaration
function squareDecl(n) { return n * n; }

// expression
const squareExpr = function(n) { return n * n; };
// declarations are hoisted; expressions are not.
```

**Expected output:**
Three pairs, each with a one-line comment.

### Task 3: First DOM page

**What to do:**
Create `js-lab/week4-day1/index.html` with:
- A `<h1>` with id `title`
- A `<p>` with id `message`
- Three `<button>` elements with ids `btn-shout`, `btn-reset`, `btn-color`

Link `main.js` at the bottom of `<body>`.

In `main.js`, write code that:
1. Selects each element using `document.getElementById`
2. On `btn-shout` click, sets the message to "HELLO WORLD" in uppercase
3. On `btn-reset` click, sets the message back to "Welcome"
4. On `btn-color` click, toggles a class `highlight` on the title (define `.highlight { color: red; }` in a `<style>` block in the head)

**Expected output:**
A page with a title, message, and three buttons. Clicking each button changes the page visibly.

### Task 4: Use querySelector and querySelectorAll

**What to do:**
Add three more buttons (`<button class="action">`, three of them) to your HTML. Use `document.querySelectorAll(".action")` to grab them all, then use `forEach` to attach a click listener to each that logs which one was clicked.

```javascript
const actions = document.querySelectorAll(".action");
actions.forEach((btn, i) => {
  btn.addEventListener("click", () => {
    console.log(`Action button ${i + 1} clicked`);
  });
});
```

**Expected output:**
Three buttons, clicking each one logs the index.

### Task 5: Build-first log entry

**What to do:**
Pick one function from Task 1 (say, `countVowels`) and add a stretch feature to it -- for example, also count consonants. The rule is:

1. First, write the extension yourself by hand.
2. Then, ask AI for an alternative version.
3. In `AI_AUDIT.md`, record:
   - Your pseudocode (3-5 lines)
   - Your manual working version (link to commit)
   - Your prompt to AI
   - AI's version
   - What you kept and what you rejected

**Expected output:**
A complete Build-first log entry in `AI_AUDIT.md`.

## Stretch Goals (Optional - Extra Credit)

- Add a fourth button that uses `setInterval` to update a counter on the page once per second.
- Use `addEventListener` with the `{ once: true }` option to make a button only fire once.
- Write a function `debounce(fn, ms)` and use it on a fast-firing button. (Hard.)

## Submission Requirements

- **What to submit:** Repo link, `js-lab/week4-day1/` folder, `AI_AUDIT.md` updated.
- **Where:** Week 4 submissions channel
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| 10 functions correct | 25 | All 10 functions defined and tested. Correct return values for the test cases. |
| Declaration vs expression notes | 10 | Three function pairs with explanatory comments. |
| DOM page with three buttons | 20 | All three buttons work as specified. Class toggle works. |
| querySelectorAll forEach | 10 | All three action buttons logging correctly. |
| Build-first log in audit | 20 | Full entry: pseudocode, manual version, prompt, AI version, decisions. |
| Clean commits | 10 | Conventional messages, three-plus commits. |
| Code quality | 5 | `const` by default, no `var`, comments where helpful. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Forgetting to load JS at the bottom of `<body>`.** If `<script>` is in `<head>`, the JS runs before the DOM exists and `document.getElementById` returns `null`. Either load at the bottom or use `defer` on the script tag.
- **Reassigning a function declaration.** Declarations are not constants; you can accidentally clobber one. Prefer `const fn = ...` for safety.
- **Not testing event listeners after wiring them.** Click every button at least once before committing.

## Resources

- Day 1 readings: [Function Declarations & Expressions.md](./Function%20Declarations%20&%20Expressions.md), [JavaScript-DOM.md](./JavaScript-DOM.md)
- Week 4 AI boundaries: [../ai.md](../ai.md)
- MDN addEventListener: https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener
