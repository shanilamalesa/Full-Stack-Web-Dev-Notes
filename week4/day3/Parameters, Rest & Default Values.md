# Parameters, Rest & Default Values

## Learning Objectives
By the end of this lesson, you will be able to:
- Clearly distinguish between a parameter and an argument.
- Define default values for parameters to handle missing arguments gracefully.
- Use rest parameters (`...`) to collect any number of arguments into a real array.
- Use the spread operator (`...`) to expand arrays or objects into function calls or literals.
- Recognise the difference between rest (in function definitions) and spread (in calls and literals).

---

## 1. Parameters vs. Arguments: What Is the Difference?

These two words are often used interchangeably, but they refer to different things. Understanding the distinction prevents confusion when reading documentation or error messages.

**A parameter** is the name you define in the function signature - the placeholder that waits for a value to be given.

**An argument** is the actual value you pass in when you call the function.

```javascript
// 'name' and 'age' are PARAMETERS - placeholders defined in the signature
function introduce(name, age) {
  console.log(`My name is ${name} and I am ${age} years old.`);
}

// 'Alice' and 30 are ARGUMENTS - real values passed at the point of calling
introduce('Alice', 30);
```

### An Analogy

Think of a function as a recipe card. The parameters are the ingredients listed on the card: "flour", "eggs", "butter". They describe what the recipe needs but are not the real items themselves.

When you actually cook the dish, you bring real flour from the pantry, real eggs from the fridge. Those real items are the arguments. The recipe (function) does not change - only the specific ingredients you bring each time may differ.

| Term | Where it lives | Example |
|---|---|---|
| Parameter | Inside the function definition | `function greet(name)` |
| Argument | At the point of calling the function | `greet('Alice')` |

---

## 2. Default Parameter Values

### The Problem: Missing Arguments Become `undefined`

When you call a function with fewer arguments than it has parameters, the missing parameters receive the value `undefined`. This can cause unexpected output or runtime errors.

**Before defaults - the problem:**

```javascript
// No default values defined
function greet(name, greeting) {
  console.log(`${greeting}, ${name}!`);
}

greet('Alice');
// Logs: "undefined, Alice!"
// 'greeting' was never passed, so it is undefined
```

### The Solution: Default Parameter Values

You can assign a fallback value directly in the parameter list using `=`. This value is used whenever the argument is `undefined` - either because it was not passed at all, or because `undefined` was explicitly passed.

**After defaults - the fix:**

```javascript
function greet(name = 'Guest', greeting = 'Hello') {
  console.log(`${greeting}, ${name}!`);
}

greet('Alice');              // "Hello, Alice!"  - greeting uses its default
greet();                     // "Hello, Guest!"  - both use their defaults
greet('Bob', 'Good morning'); // "Good morning, Bob!" - no defaults needed
greet(undefined, 'Hi');      // "Hi, Guest!"    - undefined triggers the default
```

### Important Nuance: Only `undefined` Triggers the Default

Passing `null`, `0`, `false`, or an empty string `''` does **not** trigger a default. Those are real values. Only `undefined` (explicitly or through omission) triggers it.

```javascript
function display(value = 'nothing') {
  console.log(value);
}

display(null);  // logs null  - default NOT triggered
display(0);     // logs 0     - default NOT triggered
display('');    // logs ''    - default NOT triggered
display();      // logs 'nothing' - default IS triggered
```

### Defaults Can Be Expressions

Default values are not limited to simple literals. They can be the result of any expression, including function calls:

```javascript
function rollDice(sides = 6) {
  return Math.floor(Math.random() * sides) + 1;
}

rollDice();   // rolls a standard 6-sided die
rollDice(20); // rolls a 20-sided die
```

---

## 3. Rest Parameters: Gathering Arguments into an Array

### The Problem They Solve

Sometimes you want a function to accept any number of arguments - you do not know in advance exactly how many the caller will pass. Before ES6, developers had to rely on the `arguments` object, which is array-like but not a real array.

```javascript
// Old approach using 'arguments' (not recommended in new code)
function sumOld() {
  let total = 0;
  for (let i = 0; i < arguments.length; i++) {
    total += arguments[i];
  }
  return total;
}
```

The problem is that `arguments` is not a real `Array`. You cannot call `.map()`, `.filter()`, or `.reduce()` directly on it without converting it first.

### The Solution: Rest Parameters (`...`)

The **rest parameter** syntax uses three dots (`...`) before a parameter name. It gathers all remaining arguments passed to the function into a **real `Array`**, giving you full access to every array method.

```javascript
// 'nums' will be a genuine Array containing all passed values
function sumAll(...nums) {
  console.log(nums);        // e.g. [1, 2, 3, 4]
  console.log(Array.isArray(nums)); // true
  return nums.reduce((total, n) => total + n, 0);
}

console.log(sumAll(1, 2, 3));         // 6
console.log(sumAll(10, 20, 30, 40));  // 100
console.log(sumAll());                // 0 (empty array, reduce returns initial value)
```

The `...nums` syntax tells JavaScript: "take every argument from this point onward and collect them into an array called `nums`."

### The Golden Rule: Rest Must Be Last

A rest parameter must always be the **last parameter** in the function signature. This is because it collects everything that is left over. Having parameters after a rest parameter would be ambiguous - JavaScript would not know which arguments belong to rest and which to the ones that follow.

```javascript
// CORRECT: named parameter first, rest collects the remainder
function joinWith(separator, ...items) {
  return items.join(separator);
}

console.log(joinWith('-', 'a', 'b', 'c')); // "a-b-c"
console.log(joinWith(', ', 'one', 'two')); // "one, two"
```

```javascript
// WRONG: rest parameter is not last - this is a SyntaxError
function broken(...items, separator) { // SyntaxError
  return items.join(separator);
}
```

### How the Gathering Logic Works

When JavaScript processes a call to a function with rest parameters, it assigns arguments to named parameters first (left to right), and then collects **all remaining arguments** into the rest array:

```javascript
function example(first, second, ...rest) {
  console.log('first:', first);   // 1
  console.log('second:', second); // 2
  console.log('rest:', rest);     // [3, 4, 5]
}

example(1, 2, 3, 4, 5);
```

If no extra arguments are passed, the rest parameter becomes an empty array - never `undefined`:

```javascript
function example(first, ...rest) {
  console.log('rest:', rest); // []
}

example('only one argument');
```

---

## 4. The Spread Operator: Expanding Arrays and Objects

The spread operator uses the same three-dot syntax (`...`) but does the opposite of rest. While rest **gathers** individual values into an array, spread **expands** an array (or any iterable) into individual values.

Whether `...` means rest or spread depends on context:
- In a **function definition** - it is rest (gathering).
- In a **function call or a literal** - it is spread (expanding).

### Spreading in Function Calls

Without spread, you cannot pass an array directly to a function that expects individual arguments:

```javascript
const numbers = [3, 1, 4, 1, 5, 9];

// Without spread: does not work as expected
console.log(Math.max(numbers)); // NaN - Math.max cannot handle an array

// With spread: expands the array into individual arguments
console.log(Math.max(...numbers)); // 9
```

### Spreading into a New Array

You can use spread to copy an array or build a new one from existing arrays:

```javascript
const original = [1, 2, 3];

// Create a copy (a new array - not the same reference)
const copy = [...original];
console.log(copy);          // [1, 2, 3]

// Combine arrays
const extended = [...original, 4, 5];
console.log(extended);      // [1, 2, 3, 4, 5]

// Merge two arrays
const a = [1, 2];
const b = [3, 4];
const merged = [...a, ...b];
console.log(merged);        // [1, 2, 3, 4]
```

### Spreading into a New Object

Object spread works the same way, creating a shallow clone:

```javascript
const defaults = { theme: 'light', language: 'en', fontSize: 14 };
const userSettings = { theme: 'dark', fontSize: 18 };

// Later properties overwrite earlier ones
const finalSettings = { ...defaults, ...userSettings };
console.log(finalSettings);
// { theme: 'dark', language: 'en', fontSize: 18 }
```

This is the same idea as `Object.assign`, but cleaner to read and does not mutate any existing object.

---

## 5. Rest vs. Spread at a Glance

| | Rest | Spread |
|---|---|---|
| Syntax | `...paramName` in a function definition | `...iterable` in a call or literal |
| What it does | Gathers remaining arguments into an array | Expands an array or object into individual items |
| Position rule | Must be the last parameter | No position restriction |
| Example | `function f(...args)` | `Math.max(...nums)` or `[...arr]` |

---

## 6. Putting It All Together

This example combines default parameters, rest, and spread in a single function:

```javascript
function createUser(username, {
  role = 'guest',        // default for role
  active = true,         // default for active
  theme = 'light',       // default for theme
  ...extras              // rest: collect any additional settings
} = {}) {                // default the entire second parameter to empty object

  return {
    username,
    role,
    active,
    theme,
    ...extras            // spread extras into the returned object
  };
}

// Example 1: no config passed at all
console.log(createUser('alice'));
// { username: 'alice', role: 'guest', active: true, theme: 'light' }

// Example 2: some overrides, plus an extra property
console.log(createUser('bob', { role: 'admin', theme: 'dark', age: 30 }));
// { username: 'bob', role: 'admin', active: true, theme: 'dark', age: 30 }
```

What each part does:
- The second parameter defaults to `{}` so that calling `createUser('alice')` without a config object does not throw an error.
- `role`, `active`, and `theme` are destructured with defaults from whatever config is passed.
- `...extras` collects any unrecognised properties (like `age`) so they are not silently dropped.
- `...extras` in the `return` spreads those extra properties into the returned object.

---

## 7. In-Class Activity: Parameter Utilities

### Part A: `mergeOptions`

Write a function `mergeOptions(defaults, overrides)` that:
- Takes two objects.
- Returns a new merged object where properties from `overrides` replace those in `defaults`.
- Uses object spread for both cloning and merging - do not mutate either input.

```javascript
const defaults = { a: 1, b: 2 };
const overrides = { b: 42, c: 3 };

console.log(mergeOptions(defaults, overrides));
// { a: 1, b: 42, c: 3 }

console.log(defaults); // { a: 1, b: 2 } - unchanged
```

### Part B: `logAll`

Write a function `logAll(prefix, ...items)` that:
- Logs each item to the console, prefixed by the given string.
- Uses the rest parameter for items.

```javascript
logAll('ITEM:', 'apple', 'banana', 'cherry');
// ITEM: apple
// ITEM: banana
// ITEM: cherry

logAll('VALUE:', 42);
// VALUE: 42

logAll('EMPTY:');
// (no output - rest is an empty array)
```

---

## 8. Common Beginner Mistakes

| Mistake | What Goes Wrong | Fix |
|---|---|---|
| Placing rest before other parameters | `SyntaxError` - rest must be last. | Move `...rest` to the end of the parameter list. |
| Expecting `null` or `0` to trigger a default | The default only fires for `undefined`. | Use a conditional inside the function if you need to catch falsy values. |
| Treating `arguments` like a real array | `arguments` has no `.map()`, `.filter()` etc. | Use rest parameters instead. |
| Mutating the spread source | Spread creates a shallow copy - nested objects are still shared references. | Deep clone if needed, or use structured cloning for nested data. |
| Confusing rest and spread | Both use `...` but do opposite things. | Ask: am I in a definition (rest/gathering) or a call/literal (spread/expanding)? |

---

## 9. Assignment: Flexible Calculator

Build a function `flexCalc` with the following behaviour:

- The first argument is an operator string: `'+'`, `'-'`, `'*'`, or `'/'`.
- All remaining arguments are numbers, collected using a rest parameter.
- The function applies the operator cumulatively across all the numbers.

```javascript
console.log(flexCalc('+', 1, 2, 3, 4));  // 10   (1 + 2 + 3 + 4)
console.log(flexCalc('*', 2, 3, 4));      // 24   (2 * 3 * 4)
console.log(flexCalc('-', 10, 3, 2));     // 5    (10 - 3 - 2)
console.log(flexCalc('/', 100, 5, 4));    // 5    (100 / 5 / 4)
```

### Edge Cases to Handle

- No numbers passed: return `0` for `'+'`, return `1` for `'*'`, return `null` for others.
- Division by zero: return the string `'Error: division by zero'`.

### Stretch Goal

Refactor `flexCalc` so it accepts its numbers as a pre-built array using the spread operator at the call site:

```javascript
const values = [2, 3, 4];
console.log(flexCalc('*', ...values)); // 24
```

Your function signature should not change - the spread happens when calling the function, not inside it.

---

## Summary

| Concept | Key Point |
|---|---|
| Parameter | The placeholder name in a function definition |
| Argument | The actual value passed when the function is called |
| Default value | Assigned with `=` in the definition; only triggered when the argument is `undefined` |
| Rest parameter | Collects remaining arguments into a real array; must be last |
| Spread (array) | Expands an array into individual values in a call or literal |
| Spread (object) | Copies properties from one object into another |
| Rest vs spread | Same syntax, opposite directions: rest gathers, spread expands |

With default parameters, rest gathering, and spread expansion solid, you are ready for the next lesson on closures and scope chains.