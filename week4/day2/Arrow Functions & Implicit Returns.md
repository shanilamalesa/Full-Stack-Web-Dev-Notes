# Arrow Functions & Implicit Returns

## Learning Objectives
By the end of this lesson, you will be able to:
- Write functions using the ES6 arrow (`=>`) syntax.
- Transform a standard function declaration into an arrow function step by step.
- Use implicit returns correctly for single-expression functions.
- Understand the specific rules about when curly braces must be removed or kept.
- Explain how arrow functions handle `this` (lexical binding).
- Recognize when you should not use an arrow function.

---

## 1. What Is an Arrow Function?

An arrow function is a shorter way to write a function expression. It was introduced in ES6 (2015) and is now the most common function syntax you will see in modern JavaScript, especially inside callbacks.

Arrow functions are not a replacement for all functions. They have specific behaviours - particularly around `this` - that make them the right choice in some situations and the wrong choice in others. You will learn both sides in this lesson.

---

## 2. Step-by-Step Transformation

The best way to understand arrow function syntax is to start with a regular function and convert it one step at a time. Each step removes something, making the function shorter, until you arrive at the final compact form.

We will use a simple function that doubles a number as the working example.

### Step 1: Start with a Function Declaration

This is the standard starting point you already know:

```javascript
function double(n) {
  return n * 2;
}
```

### Step 2: Convert to a Function Expression

Remove the name from the `function` keyword and assign it to a variable:

```javascript
const double = function(n) {
  return n * 2;
};
```

The behaviour is identical. The difference is that this is now a function expression - a function stored in a variable.

### Step 3: Replace `function` with an Arrow

Remove the `function` keyword entirely and add `=>` (the "fat arrow") between the parameters and the opening brace:

```javascript
const double = (n) => {
  return n * 2;
};
```

This is a valid arrow function. The `{}` body and `return` keyword still work exactly as before.

### Step 4: Remove the Parentheses Around a Single Parameter

When there is exactly one parameter, you may drop the parentheses:

```javascript
const double = n => {
  return n * 2;
};
```

This is a matter of style. Some teams always keep the parentheses for consistency. Either form is correct.

### Step 5: Use an Implicit Return (the Final Short Form)

When the entire function body is a single expression that you want to return, you can remove both the curly braces and the `return` keyword. JavaScript will return the expression automatically:

```javascript
const double = n => n * 2;
```

This is the shortest possible form. The expression `n * 2` is implicitly returned - you do not write `return`.

All five versions produce exactly the same result:

```javascript
console.log(double(5)); // 25
```

---

## 3. The Rules of Implicit Returns

Implicit returns are useful, but they have precise rules. Breaking these rules is one of the most common sources of bugs when writing arrow functions.

### Rule 1: No Curly Braces Means Implicit Return

When you write an arrow function without curly braces, the expression after `=>` is automatically returned:

```javascript
// This returns n * 2 automatically
const double = n => n * 2;
```

### Rule 2: Curly Braces Require an Explicit `return`

The moment you add curly braces, you are writing a block body. JavaScript treats this the same as a normal function body. You must write `return` yourself, or the function returns `undefined`:

```javascript
// With curly braces: you MUST write return
const double = n => {
  return n * 2;  // explicit return required
};

// With curly braces and no return: function returns undefined!
const broken = n => {
  n * 2; // this runs, but the result is discarded
};

console.log(broken(5)); // undefined
```

### Rule 3: To Return an Object Literal, Wrap It in Parentheses

This is the most common implicit return mistake. An object literal uses curly braces `{}`, and JavaScript cannot tell whether you mean "start a block body" or "start an object". The solution is to wrap the object in parentheses:

```javascript
// WRONG: JavaScript reads the {} as a block body, not an object
const makeUser = (name, age) => { name, age }; // syntax error or returns undefined

// CORRECT: Parentheses tell JavaScript this is an object expression
const makeUser = (name, age) => ({ name, age });

console.log(makeUser('Alice', 30)); // { name: 'Alice', age: 30 }
```

### Rule 4: Multi-Line Logic Always Requires Curly Braces

If your function has more than one line of logic, you must use curly braces and an explicit return:

```javascript
// Multi-line requires a block body
const processNumber = n => {
  const doubled = n * 2;
  const formatted = `Result: ${doubled}`;
  return formatted;
};

console.log(processNumber(5)); // "Result: 10"
```

---

## 4. Syntax Quick Reference

| Situation | Example |
|---|---|
| Zero parameters | `() => expression` |
| One parameter | `n => expression` or `(n) => expression` |
| Multiple parameters | `(a, b) => expression` |
| Single-expression body (implicit return) | `n => n * 2` |
| Multi-line body (explicit return required) | `n => { const x = n * 2; return x; }` |
| Returning an object literal | `(name) => ({ name })` |

---

## 5. Common Arrow Function Examples

Arrow functions are most frequently used as inline callbacks. Here are the typical patterns you will see in real code:

### With `.map()`

```javascript
const numbers = [1, 2, 3, 4, 5];

// Longhand (function expression)
const doubled = numbers.map(function(n) {
  return n * 2;
});

// Arrow function, implicit return
const doubled = numbers.map(n => n * 2);

console.log(doubled); // [2, 4, 6, 8, 10]
```

### With `.filter()`

```javascript
const numbers = [1, 2, 3, 4, 5];

// Only keep numbers greater than 2
const large = numbers.filter(n => n > 2);

console.log(large); // [3, 4, 5]
```

### With `.reduce()`

```javascript
const numbers = [1, 2, 3, 4, 5];

// Sum all numbers
const total = numbers.reduce((accumulator, n) => accumulator + n, 0);

console.log(total); // 15
```

### Chained

```javascript
const numbers = [1, 2, 3, 4, 5];

// Get doubled even numbers
const result = numbers
  .filter(n => n % 2 === 0)   // [2, 4]
  .map(n => n * 2);            // [4, 8]

console.log(result); // [4, 8]
```

---

## 6. Lexical `this` - How Arrow Functions Handle Context

This is the most important behavioural difference between arrow functions and regular functions.

Regular functions create their own `this` value depending on how they are called. This causes a well-known problem inside callbacks: `this` inside the callback ends up pointing to something different from what you intended.

Arrow functions do not have their own `this`. They **inherit** `this` from the surrounding code at the time they are defined. This is called **lexical `this`** - the value of `this` is determined by where the arrow function is written, not how it is called.

### The Problem with Regular Functions

```javascript
function Timer() {
  this.seconds = 0;

  // Regular function callback: 'this' no longer refers to the Timer instance
  setInterval(function() {
    this.seconds++; // 'this' here is the global object, not the Timer
    console.log(this.seconds); // NaN or unintended behaviour
  }, 1000);
}

new Timer();
```

The classic fix before arrow functions was to save `this` in another variable:

```javascript
function Timer() {
  this.seconds = 0;
  const self = this; // save reference

  setInterval(function() {
    self.seconds++; // works, but awkward
    console.log(self.seconds);
  }, 1000);
}
```

### The Arrow Function Solution

```javascript
function Timer() {
  this.seconds = 0;

  // Arrow function: 'this' is inherited from Timer
  setInterval(() => {
    this.seconds++; // 'this' correctly refers to the Timer instance
    console.log(this.seconds); // 1, 2, 3, ...
  }, 1000);
}

new Timer();
```

The arrow function inherits `this` from the `Timer` function that wraps it. No need for `const self = this`.

---

## 7. When Not to Use Arrow Functions

Because arrow functions do not have their own `this`, they are the wrong choice in specific situations.

### Object Methods

When a function is used as a method on an object, it typically needs `this` to refer to that object. Arrow functions are unsuitable here:

```javascript
const counter = {
  count: 0,

  // WRONG: arrow function has no own 'this', so 'this' is not the object
  increment: () => {
    this.count++; // 'this' is the surrounding scope (likely global), not counter
  },

  // CORRECT: regular method syntax gives you 'this' as the object
  incrementCorrect() {
    this.count++;
  }
};

counter.incrementCorrect();
console.log(counter.count); // 1
```

### Constructors

Arrow functions cannot be used with the `new` keyword. They have no `prototype` property:

```javascript
const Person = (name) => {
  this.name = name;
};

const p = new Person('Alice'); // TypeError: Person is not a constructor
```

Use a regular function declaration or a class for constructors.

### When You Need the `arguments` Object

Arrow functions do not have their own `arguments` object. If you need it, use a regular function - or better, use rest parameters:

```javascript
// This will throw a ReferenceError inside an arrow function
const logArgs = () => console.log(arguments); // ReferenceError

// Use rest parameters instead
const logArgs = (...args) => console.log(args); // works correctly
```

### Summary: Arrow vs Regular Function

| Situation | Use Arrow | Use Regular |
|---|---|---|
| Array callbacks (`.map`, `.filter`, `.reduce`) | Yes | No |
| Timers and async callbacks | Yes | No |
| Object methods | No | Yes |
| Constructors | No | Yes |
| Needing the `arguments` object | No | Yes (or use rest) |
| Top-level utility functions | Either | Either |

---

## 8. In-Class Activity: Refactor Array Callbacks

You are given the following code using regular function expressions. Refactor each callback to use an arrow function with an implicit return where possible.

### Starting Code

```javascript
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map(function(n) {
  return n * 2;
});

const evens = numbers.filter(function(n) {
  return n % 2 === 0;
});

console.log(doubled); // [2, 4, 6, 8, 10]
console.log(evens);   // [2, 4]
```

### Tasks

1. Rewrite the `.map()` and `.filter()` callbacks as concise arrow functions.
2. Chain them together to get doubled even numbers (expected result: `[4, 8]`):

```javascript
const doubledEvens = /* your solution here */;
console.log(doubledEvens); // [4, 8]
```

3. Add a `.reduce()` at the end of that chain to sum the doubled evens (expected result: `12`).

---

## 9. Common Beginner Mistakes

| Mistake | What Goes Wrong |
|---|---|
| Using curly braces but forgetting `return` | Function returns `undefined` silently. |
| Returning an object literal without parentheses | JavaScript reads `{}` as a block, not an object. Wrap with `()`. |
| Using an arrow function as an object method | `this` will not refer to the object. Use a regular method. |
| Using an arrow function as a constructor with `new` | Throws a `TypeError`. Arrow functions have no prototype. |
| Expecting `arguments` to work in an arrow function | Arrow functions inherit `arguments` from outside, which may be wrong or unavailable. Use rest parameters. |

---

## 10. Assignment: Extract Emails with Arrow Functions

### Data

```javascript
const users = [
  { id: 1, name: 'Alice',   email: 'alice@example.com' },
  { id: 2, name: 'Bob',     email: 'bob@example.org' },
  { id: 3, name: 'Charlie', email: 'charlie@mail.net' }
];
```

### Part 1: Extract All Emails

Use `.map()` with a concise arrow function to produce an array of email strings. Your solution should fit on one line:

```javascript
const emails = users.map(/* your arrow function */);
console.log(emails);
// ['alice@example.com', 'bob@example.org', 'charlie@mail.net']
```

### Part 2: Filter Before Mapping

Chain a `.filter()` before your `.map()` to include only users whose email ends with `@example.com`:

```javascript
const exampleEmails = users
  .filter(/* your arrow function */)
  .map(/* your arrow function */);

console.log(exampleEmails);
// ['alice@example.com']
```

### Stretch Goal

Using the same `users` array, produce an array of user objects that contain only `id` and `name` (drop the `email` property). Use `.map()` and an implicit return of an object literal:

```javascript
const stripped = users.map(/* your arrow function returning { id, name } */);
console.log(stripped);
// [{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }, { id: 3, name: 'Charlie' }]
```

---

## Summary

| Concept | Key Rule |
|---|---|
| Arrow syntax | Replace `function(params)` with `(params) =>` |
| No curly braces | Expression is returned implicitly |
| With curly braces | `return` is required, or the function returns `undefined` |
| Returning an object literal | Must wrap in parentheses: `() => ({ key: value })` |
| Lexical `this` | Arrow functions inherit `this` from the surrounding scope |
| Object methods | Do not use arrow functions - use regular method syntax |
| Constructors | Do not use arrow functions - they cannot be called with `new` |

With arrow functions and implicit returns mastered, you are ready for the next lesson on default parameters and rest syntax.