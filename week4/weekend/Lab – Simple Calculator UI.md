# Weekend Lab: Simple Calculator UI

## Project Overview

In this lab you will build a fully functional, browser-based calculator using HTML, CSS, and JavaScript. The goal is not just to make the calculator work - it is to practise writing clean, well-organised code by keeping your logic, your DOM manipulation, and your styling in separate, focused layers.

By the end of this lab you will have applied:
- Writing and reusing a pure logic function.
- Reading and writing DOM elements.
- Attaching event listeners to multiple buttons efficiently.
- Validating user input before doing any computation.
- Separating concerns: logic lives in one function, DOM updates live in another, event handling lives in a third.

---

## Learning Objectives

- Integrate custom functions with DOM manipulation.
- Wire up event listeners to UI controls.
- Maintain a clear separation between computation logic and presentation.
- Handle user input validation and provide meaningful error feedback.

---

## Part 1: Project Structure

Before writing any code, set up your project files:

```
week4/weekend/calculator/
  calculator.html
  calculator.js
  calculator.css
```

You will work exclusively in these two files. The HTML file provides the interface; the JavaScript file provides all the behaviour.

---

## Part 2: The HTML Structure

Create `calculator.html` with the following markup. Read through the comments carefully - each decision is explained.
Separate the HTML from the CSS.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Simple Calculator</title>
  <style>
    /* Base layout */
    body {
      font-family: sans-serif;
      background: #f4f4f4;
      display: flex;
      justify-content: center;
      padding: 2rem;
    }

    .calc-container {
      background: white;
      border-radius: 8px;
      padding: 2rem;
      max-width: 320px;
      width: 100%;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }

    h2 {
      margin-top: 0;
    }

    .input-group {
      display: flex;
      flex-direction: column;
      gap: 0.5rem;
      margin-bottom: 1rem;
    }

    input {
      font-size: 1rem;
      padding: 0.5rem 0.75rem;
      border: 1px solid #ccc;
      border-radius: 4px;
      width: 100%;
      box-sizing: border-box;
    }

    .operator-buttons {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 0.5rem;
      margin-bottom: 1rem;
    }

    button {
      font-size: 1.25rem;
      padding: 0.6rem;
      cursor: pointer;
      border: 1px solid #ccc;
      border-radius: 4px;
      background: #f9f9f9;
    }

    button:hover {
      background: #e8e8e8;
    }

    #result {
      font-size: 1.1rem;
      font-weight: bold;
      min-height: 1.5rem;
      padding: 0.5rem;
      border-radius: 4px;
      background: #fafafa;
      border: 1px solid #eee;
    }

    /* Applied when the result is an error message */
    #result.error {
      color: #cc0000;
      background: #fff5f5;
      border-color: #ffcccc;
    }
  </style>
</head>
<body>
  <div class="calc-container">
    <h2>Simple Calculator</h2>

    <!-- Input fields for the two operands -->
    <div class="input-group">
      <input id="num1" type="text" placeholder="First number" />
      <input id="num2" type="text" placeholder="Second number" />
    </div>

    <!-- Operator buttons - each stores its operator in a data-op attribute -->
    <div class="operator-buttons">
      <button data-op="+">+</button>
      <button data-op="-">-</button>
      <button data-op="*">x</button>
      <button data-op="/">/</button>
    </div>

    <!-- Result is written here by JavaScript -->
    <div id="result"></div>
  </div>

  <!-- Load the JavaScript file after the HTML elements exist -->
  <script src="calculator.js"></script>
</body>
</html>
```

### Key HTML Decisions Explained

**`data-op` attributes**: Instead of writing four separate event handlers - one for each button - you store the operator as a data attribute directly on the button. JavaScript can then read this value from whichever button was clicked, letting you handle all four buttons with a single function.

**`<div id="result">`**: This element starts empty. JavaScript will write text into it after each calculation. When there is an error, the `.error` CSS class is added to change its appearance. When there is a valid result, the class is removed.

**Script tag at the bottom**: The `<script>` tag is placed at the end of the `<body>`. This ensures all the HTML elements above it already exist in the DOM by the time the JavaScript tries to query them.

---

## Part 3: The Calculator Logic Function

Open `calculator.js`. Start with the pure logic function. This function has no knowledge of the DOM - it only receives values and returns a result.

```javascript
// calculator.js

/**
 * Performs basic arithmetic on two numbers.
 *
 * @param {number} a - The first operand.
 * @param {string} operator - One of '+', '-', '*', '/'.
 * @param {number} b - The second operand.
 * @returns {number|string} The numeric result, or an error message string.
 */
function calculate(a, operator, b) {
  switch (operator) {
    case '+':
      return a + b;
    case '-':
      return a - b;
    case '*':
      return a * b;
    case '/':
      if (b === 0) {
        return 'Error: Cannot divide by zero';
      }
      return a / b;
    default:
      return 'Error: Unknown operator';
  }
}
```

### Why Write Logic Separately?

This function does one job and does it well. It does not touch the DOM, does not read any input fields, and does not know anything about buttons or HTML. This makes it:
- Easy to test in isolation (open the browser console and call `calculate(10, '+', 5)` directly).
- Easy to reuse in other parts of the application.
- Easy to read and debug - there are no UI concerns mixed in.

This principle is called **separation of concerns**.

---

## Part 4: Selecting DOM Elements

After the logic function, select and cache references to the DOM elements you will interact with. Do this once, at load time, rather than querying the DOM on every button click.

```javascript
// Grab the input fields
const input1 = document.getElementById('num1');
const input2 = document.getElementById('num2');

// Grab all four operator buttons using a CSS attribute selector
const operatorButtons = document.querySelectorAll('button[data-op]');

// Grab the result display element
const resultDisplay = document.getElementById('result');
```

### Why Cache DOM References?

Every call to `document.getElementById` or `document.querySelector` asks the browser to search through the entire document. If you call these inside an event handler, they run on every single click. Saving the references to variables means the search happens once and the results are reused - this is called **DOM caching**.

---

## Part 5: The Display Helper Function

Write a small helper function that handles all updates to the result element. Keeping DOM updates in one location means you only have one place to change if the output format ever needs to be different.

```javascript
/**
 * Writes a message into the result display element.
 * Applies or removes the error style depending on the isError flag.
 *
 * @param {string|number} message - The text or number to display.
 * @param {boolean} isError - Whether the message represents an error.
 */
function displayResult(message, isError = false) {
  resultDisplay.textContent = message;

  // Add the 'error' class if isError is true; remove it if false
  resultDisplay.classList.toggle('error', isError);
}
```

`classList.toggle(className, condition)` is a convenient method: when the second argument is `true`, it adds the class; when `false`, it removes it. This replaces the need for separate `add` and `remove` calls.

---

## Part 6: The Event Handler

Write the function that runs whenever any operator button is clicked. This function reads the inputs, validates them, calls `calculate`, and updates the display.

```javascript
/**
 * Handles a click on any operator button.
 * Reads input values, validates them, computes the result, and displays it.
 *
 * @param {MouseEvent} event - The click event from the browser.
 */
function handleOperatorClick(event) {
  // Read which operator was clicked from the button's data-op attribute
  const operator = event.target.dataset.op;

  // Read raw string values from both input fields and remove leading/trailing whitespace
  const raw1 = input1.value.trim();
  const raw2 = input2.value.trim();

  // If either field is empty, show an error immediately and stop
  if (raw1 === '' || raw2 === '') {
    displayResult('Please fill in both number fields', true);
    return;
  }

  // Convert the strings to numbers
  const a = Number(raw1);
  const b = Number(raw2);

  // If either conversion failed (non-numeric text was entered), show an error
  if (Number.isNaN(a) || Number.isNaN(b)) {
    displayResult('Please enter valid numbers only', true);
    return;
  }

  // All input is valid - compute the result
  const result = calculate(a, operator, b);

  // If calculate returned a string, it is an error message
  const isError = typeof result === 'string';

  displayResult(result, isError);
}
```

### Why Validate Input?

Users do not always type what you expect. They may leave a field blank, type a word, paste extra spaces, or enter a special character. Validating input before calling `calculate` means your logic function always receives clean, usable values, and the user always receives clear feedback instead of a confusing `NaN` on screen.

Two checks are used here:
1. **Empty check**: `raw1 === ''` catches blank inputs before any conversion.
2. **`Number.isNaN` check**: `Number('abc')` returns `NaN`. `Number.isNaN` reliably detects this. Note that `isNaN('abc')` would also work here but `Number.isNaN` is more precise.

---

## Part 7: Attaching the Event Listeners

The final step is wiring the event handler to all four buttons. Because `querySelectorAll` returns a NodeList, you can use `forEach` to loop through it and attach the same handler to each button.

```javascript
// Attach the click handler to every operator button
operatorButtons.forEach(function(button) {
  button.addEventListener('click', handleOperatorClick);
});
```

Or with an arrow function:

```javascript
operatorButtons.forEach(button => button.addEventListener('click', handleOperatorClick));
```

Both are correct. The key point is that you pass `handleOperatorClick` as a reference - without calling it. The browser will call it later when a button is clicked.

---

## Part 8: The Complete `calculator.js` File

Here is the entire JavaScript file assembled in one place for reference:

```javascript
// calculator.js

// --- DOM Element References ---

const input1 = document.getElementById('num1');
const input2 = document.getElementById('num2');
const operatorButtons = document.querySelectorAll('button[data-op]');
const resultDisplay = document.getElementById('result');

// --- Pure Logic Function ---

function calculate(a, operator, b) {
  switch (operator) {
    case '+': return a + b;
    case '-': return a - b;
    case '*': return a * b;
    case '/':
      if (b === 0) return 'Error: Cannot divide by zero';
      return a / b;
    default:
      return 'Error: Unknown operator';
  }
}

// --- Display Helper ---

function displayResult(message, isError = false) {
  resultDisplay.textContent = message;
  resultDisplay.classList.toggle('error', isError);
}

// --- Event Handler ---

function handleOperatorClick(event) {
  const operator = event.target.dataset.op;
  const raw1 = input1.value.trim();
  const raw2 = input2.value.trim();

  if (raw1 === '' || raw2 === '') {
    displayResult('Please fill in both number fields', true);
    return;
  }

  const a = Number(raw1);
  const b = Number(raw2);

  if (Number.isNaN(a) || Number.isNaN(b)) {
    displayResult('Please enter valid numbers only', true);
    return;
  }

  const result = calculate(a, operator, b);
  displayResult(result, typeof result === 'string');
}

// --- Attach Listeners ---

operatorButtons.forEach(button => button.addEventListener('click', handleOperatorClick));
```

---

## Part 9: Testing Your Calculator

Work through each of the following tests after completing the lab. For each test, write down what you expect before you try it, then verify the actual output.

| Input | Operator | Expected Output |
|---|---|---|
| 8, 4 | + | 12 |
| 10, 3 | - | 7 |
| 6, 7 | x | 42 |
| 20, 4 | / | 5 |
| 10, 0 | / | Error: Cannot divide by zero |
| (blank), 5 | + | Please fill in both number fields |
| abc, 5 | + | Please enter valid numbers only |
| 2.5, 4 | x | 10 |
| -10, 5 | + | -5 |
| 0, 0 | + | 0 |

---

## Part 10: Stretch Enhancements

These are optional challenges for students who finish the core lab early or want to go further.

### A. Clear Button

Add a "Clear" button to the UI that resets both input fields and removes the result text.

Hint: Add the button to the HTML and attach a click listener that sets `input1.value = ''`, `input2.value = ''`, and clears the result display.

### B. Keyboard Support

Allow the user to press the Enter key while inside either input field to trigger the last selected operator.

Hint: Listen for the `keydown` event on both inputs. Check if `event.key === 'Enter'` and store the last clicked operator in a variable that persists between calls.

### C. Calculation History Log

Append each successful calculation to a list below the result display, showing the full equation and answer.

Example output in the log:
```
8 + 4 = 12
20 / 4 = 5
6 x 7 = 42
```

Hint: Create a `<ul id="history"></ul>` in the HTML. After a successful calculation, create a new `<li>` element, set its text content, and append it to the list.

### D. Chain Results

After a calculation, populate the first input field with the result automatically, so the user can immediately use it as the starting value for the next operation.

Hint: After calling `displayResult`, also do `input1.value = result` when `result` is a number.

### E. Light and Dark Mode Toggle

Add a toggle button that switches the page between a light and a dark colour scheme.

Hint: Add a `.dark-mode` class to your CSS with alternative colours for background, text, and input elements. The button's click handler should add or remove that class on the `<body>` element.

---

## Submission Checklist

Before submitting, verify that your project meets the following criteria:

- HTML & CSS are in separate files.
- The calculator correctly computes addition, subtraction, multiplication, and division.
- Division by zero displays a clear error message.
- Blank fields display a clear error message.
- Non-numeric input displays a clear error message.
- Error messages are visually distinguished from normal results (red text or different styling).
- The JavaScript is split into clearly named functions with distinct responsibilities.
- There are no syntax errors in the browser console during normal use.
- The code file is saved in the correct directory: `week4/weekend/calculator/`.