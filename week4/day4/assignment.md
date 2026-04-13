# Week 4 - Day 4 Assignment

## Title
Closures, Scope Chains, and the Calculator UI Pre-Weekend

## Overview
Day 4 is pre-weekend polish day. Today you meet closures -- the strangest and most powerful feature of JavaScript functions -- and then apply everything from the week to begin the weekend lab: a working Simple Calculator UI. Your assignment is three closure drills plus a calculator skeleton that ships by end of day.

## Learning Objectives Assessed
- Explain what a closure is in your own words
- Use closures to create private state (counters, memoisers)
- Understand scope chains (global -> outer -> inner)
- Start the weekend Calculator Lab with a working skeleton

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio:** 75/25. **Habit:** Build first, enhance with AI. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Asking AI to explain closure behaviour after you wrote a counter example yourself.
- **NOT ALLOWED FOR:** Generating the calculator core. Explaining closures from scratch.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Closure counter

**What to do:**
Write a function `makeCounter` that returns a function which increments and returns a private count. Each call to `makeCounter` should produce an independent counter.

```javascript
function makeCounter() {
  let count = 0;
  return function() {
    count += 1;
    return count;
  };
}

const c1 = makeCounter();
const c2 = makeCounter();
console.log(c1()); // 1
console.log(c1()); // 2
console.log(c2()); // 1 -- independent
console.log(c1()); // 3
```

Type this yourself. No AI.

**Expected output:**
Working counter with the expected output sequence.

### Task 2: Closure memoiser

**What to do:**
Write a `memoise(fn)` that wraps a slow function and caches its results:

```javascript
function memoise(fn) {
  const cache = {};
  return function(arg) {
    if (cache[arg] !== undefined) return cache[arg];
    const result = fn(arg);
    cache[arg] = result;
    return result;
  };
}

const slowSquare = (n) => { console.log("computing"); return n * n; };
const fastSquare = memoise(slowSquare);

console.log(fastSquare(4)); // prints "computing", returns 16
console.log(fastSquare(4)); // does NOT print, returns 16 from cache
console.log(fastSquare(5)); // prints "computing", returns 25
```

**Expected output:**
Memoiser working. "computing" only prints on first call for each argument.

### Task 3: Scope chain sketch

**What to do:**
Create `scope-notes.md`. Draw (in text or photograph) the scope chain for this code:

```javascript
const globalVar = "I am global";

function outer() {
  const outerVar = "I am outer";
  function inner() {
    const innerVar = "I am inner";
    console.log(globalVar, outerVar, innerVar);
  }
  inner();
}
outer();
```

Label each scope level and show which variables are visible from each. 4-6 sentences of explanation in your own words.

**Expected output:**
`scope-notes.md` committed with the sketch and explanation.

### Task 4: Calculator skeleton

**What to do:**
Create `js-lab/week4-day4/calculator.html` and `calculator.js`. Build the minimum viable calculator:

- A display area (a `<div>` or `<input>` with id `display`)
- Number buttons 0-9
- Four operation buttons: +, -, \*, /
- An equals button
- A clear button

In JavaScript:
1. Select all buttons with `querySelectorAll`.
2. Maintain a state object with `firstOperand`, `operator`, and `secondOperand`.
3. On number click, append the digit to the current operand.
4. On operator click, store the operator.
5. On equals click, perform the operation and display the result.
6. On clear, reset the state.

The calculator should work for two-operand operations like `5 + 3 = 8`. Chained operations are a stretch goal.

**Expected output:**
Working calculator that can add, subtract, multiply, and divide two numbers.

### Task 5: Pre-weekend checklist

**What to do:**
Create `CHECKLIST.md`:

```markdown
## Week 4 Day 4 Pre-Weekend Checklist

- [ ] All Day 1-3 drills saved and committed
- [ ] Closure counter and memoiser working
- [ ] scope-notes.md complete
- [ ] Calculator can compute 2+3, 10-4, 6*7, 8/2 correctly
- [ ] Calculator clear button resets state
- [ ] AI_AUDIT.md has at least one build-first log per day this week
- [ ] Repo pushed and clean
```

Tick every box honestly.

**Expected output:**
`CHECKLIST.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Support chained operations: `5 + 3 + 2 = 10` without hitting equals between the additions.
- Add decimal-point support.
- Add keyboard input support using `document.addEventListener("keydown", ...)`.

## Submission Requirements

- **What to submit:** Repo with all Day 4 files, `CHECKLIST.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Closure counter | 15 | Independent counters work correctly. |
| Memoiser | 15 | Cache hit visible in the "computing" log. |
| Scope chain sketch | 10 | Labelled sketch with explanation. |
| Calculator skeleton | 30 | Add, subtract, multiply, divide all working. Clear resets. Display updates visibly. |
| Pre-weekend checklist | 10 | All boxes honestly ticked. |
| AI Audit complete | 15 | Build-first entries for each day. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Thinking a closure is a special syntax.** It is not. It is what happens whenever an inner function references an outer variable. Every function in JavaScript that references something outside itself is technically a closure.
- **Recreating `makeCounter`'s inner `count` on every call.** The whole point is that `count` lives on the outer scope and is captured by the returned function.
- **Hardcoding the calculator operator in one big if/else.** Use an object lookup instead: `const ops = { "+": (a,b) => a+b, "-": (a,b) => a-b, ... };` -- much cleaner.

## Resources

- Day 4 reading: [Closures & Scope Chains.md](./Closures%20&%20Scope%20Chains.md)
- Weekend lab: [../weekend/Lab – Simple Calculator UI.md](../weekend/Lab%20%E2%80%93%20Simple%20Calculator%20UI.md)
- Week 4 AI boundaries: [../ai.md](../ai.md)
- MDN Closures: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures
