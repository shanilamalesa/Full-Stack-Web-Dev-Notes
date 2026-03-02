# State and Events

## Learning Objectives
By the end of this lesson, you will be able to:
- Explain what state is and why React needs it.
- Use the `useState` Hook to add state to a component.
- Update state correctly using the setter function.
- Handle click events and input change events.
- Build controlled form inputs.
- Render different UI based on the current state value.
- Build simple interactive patterns like counters, toggles, and show/hide sections.

---

## 1. Why This Topic Matters

So far, the components you have built are mostly static. They can display data through props, but nothing changes when a user interacts with them. They are like a printed poster - it looks good, but it does not respond to anything.

**State** is what gives a component memory. **Events** are what let users trigger changes. When you combine the two, your interface becomes genuinely interactive.

Without state, you cannot build:
- A button that counts how many times it has been clicked.
- A password field that shows or hides its text.
- A form that disables its submit button until all fields are filled.
- A theme toggle that switches the page from light to dark mode.

State is the foundation of all of that.

---

## 2. What Is State?

State is a component's **memory**. It is a value that React stores for a component between renders. Unlike a normal JavaScript variable - which disappears as soon as the function finishes running - state persists. React keeps it alive and gives the component its current value every time the component renders.

Think of it this way:
- A normal variable is like a sticky note that you throw away after every meeting.
- State is like a whiteboard you keep in the room. You can update it, and the next time you look at it, the change is still there.

State is the right tool whenever the UI needs to change over time in response to user interaction.

---

## 3. `useState`

React provides the `useState` Hook for adding state to function components. You import it from React, call it with an initial value, and it gives you back two things: the current state value and a function to update it.

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}
```

Breaking this down:
- `count` is the current state value. You read it to display data.
- `setCount` is the setter function. You call it to update the state.
- `0` is the initial value - what `count` starts as when the component first renders.

The naming convention is `[value, setValue]`. The setter name always starts with "set" followed by the state's name. This is a convention, not a rule, but it makes code much easier to read.

---

## 4. Rules for Using `useState`

`useState` is a **Hook**, and Hooks have one important rule: they must be called at the **top level** of a function component. Never call them inside conditions, loops, or nested functions.

Correct:
```jsx
function Example() {
  const [isOpen, setIsOpen] = useState(false);
  return <div>...</div>;
}
```

Incorrect:
```jsx
function Example() {
  if (someCondition) {
    const [isOpen, setIsOpen] = useState(false); // This will break!
  }
  return <div>...</div>;
}
```

The easiest habit to build: always declare your `useState` calls at the very top of your component, before any other logic.

---

## 5. Updating State Correctly

To change a state value, you must call its setter function. You should never try to change the state variable directly.

Incorrect:
```jsx
count = count + 1; // This does nothing. React will not re-render.
```

Correct:
```jsx
setCount(count + 1); // This tells React the state changed and triggers a re-render.
```

When you call the setter function, React schedules a re-render of the component. On the next render, the component gets the new state value, and the UI updates to reflect it.

---

## 6. State Is a Snapshot

This is one of the most important and confusing concepts for beginners, so it is worth explaining clearly.

When React renders your component, it takes a **snapshot** of your state at that moment. All the event handlers that get created during that render "see" the state values from that snapshot.

What this means in practice: if you call a setter function, the state variable in the current render does not change immediately. React schedules a new render with the updated value. The new value will appear on the next render.

The mental model:
1. React renders your component. The component reads state as it currently is.
2. The user does something, such as clicking a button.
3. You call the setter function.
4. React schedules a new render.
5. On the next render, the component gets the updated state value and the UI updates.

This will save you from confusion if you ever notice that a state variable still holds its old value right after you call the setter.

---

## 7. Responding to Events

React lets you attach **event handlers** to JSX elements. An event handler is a function you write that runs when a specific user interaction happens, such as a click, a key press, or a mouse hover.

A basic example:
```jsx
function MyButton() {
  function handleClick() {
    alert('You clicked me!');
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

Key rules for event handlers in React:
- Event prop names use camelCase: `onClick`, `onChange`, `onSubmit`, `onMouseOver`.
- You pass the **function itself**, not the result of calling it.

Correct:
```jsx
<button onClick={handleClick}>Click</button>
```

Incorrect:
```jsx
<button onClick={handleClick()}>Click</button>
                              ^^
```
Adding `()` calls the function immediately when the component renders, instead of passing it as a handler to be called later on click. That is a very common beginner mistake.

---

## 8. Click Events

Click events are usually the first type of interaction you build in React. The `onClick` prop works on any built-in HTML element like `<button>`, `<div>`, or `<li>`.

A classic counter example:
```jsx
import { useState } from 'react';

function LikeButton() {
  const [likes, setLikes] = useState(0);

  function handleClick() {
    setLikes(likes + 1);
  }

  return <button onClick={handleClick}>Likes: {likes}</button>;
}
```

This demonstrates the full React interactivity loop:
1. Store data in state.
2. Respond to a user event.
3. Update state with the setter.
4. React re-renders - the UI shows the new value.

This loop is the core of almost every interactive feature you will ever build in React.

---

## 9. Input Events

Forms are where you will use state most often. React uses the `onChange` event on input fields. Every time the user types a character, `onChange` fires and you update the state.

```jsx
import { useState } from 'react';

function NameInput() {
  const [name, setName] = useState('');

  return (
    <>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter your name"
      />
      <p>Hello, {name}</p>
    </>
  );
}
```

- `e` is the event object that React passes to your handler automatically.
- `e.target.value` is the current text in the input field.
- `setName(e.target.value)` updates the state to that new text.

The result: as the user types, the paragraph below updates in real time.

---

## 10. Controlled Inputs

A **controlled input** is a form field whose value is always driven by React state. You set its `value` attribute to a state variable, and you update that state on every `onChange`.

```jsx
function EmailInput() {
  const [email, setEmail] = useState('');

  return (
    <input
      value={email}
      onChange={(e) => setEmail(e.target.value)}
    />
  );
}
```

Why this matters:
- React always knows what value the input contains.
- The UI and your state are always in sync.
- You can easily validate, preview, reset, or submit the value at any point.

This is the standard way to handle form inputs in React. You will use it everywhere.

An example with multiple controlled fields:
```jsx
import { useState } from 'react';

function RegistrationForm() {
  const [firstName, setFirstName] = useState('');
  const [email, setEmail] = useState('');

  return (
    <form>
      <input
        value={firstName}
        onChange={(e) => setFirstName(e.target.value)}
        placeholder="First name"
      />
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
    </form>
  );
}
```

---

## 11. Conditional Rendering

Once you have state, you can use it to control **what appear on screen**. Conditional rendering is the idea that different UI shows up depending on the current state value.

React uses plain JavaScript for this. There is no special React syntax.

### Using `if`
```jsx
function StatusMessage({ isLoggedIn }) {
  if (isLoggedIn) {
    return <p>Welcome back!</p>;
  }
  return <p>Please sign in.</p>;
}
```

### Using the `&&` Operator
When a condition is true, render something. When it is false, render nothing.
```jsx
function Badge({ isNew }) {
  return (
    <div>
      <span>Product</span>
      {isNew && <strong>New</strong>}
    </div>
  );
}
```

### Using a Ternary
When you need to choose between two options:
```jsx
function ThemeLabel({ darkMode }) {
  return <p>{darkMode ? 'Dark Mode' : 'Light Mode'}</p>;
}
```

These patterns become especially powerful when the condition is a piece of state, because the UI updates automatically whenever state changes.

---

## 12. Rendering Based on State

A core principle in React is that **the UI is a reflection of the current state**. You do not manually show or hide elements by reaching into the DOM. Instead, you update state, and React figures out what needs to change on screen.

This is called **declarative** programming. You declare what the UI should look like for each possible state value, and React takes care of the rest.

```jsx
import { useState } from 'react';

function OnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);

  return (
    <div>
      <p>{isOnline ? 'You are online' : 'You are offline'}</p>
      <button onClick={() => setIsOnline(!isOnline)}>
        Toggle Status
      </button>
    </div>
  );
}
```

When you click the button, `isOnline` flips between `true` and `false`, and the paragraph text updates to match automatically.

---

## 13. Toggles

A toggle is one of the best beginner patterns because it uses a simple `true`/`false` boolean value.

Common uses for toggles:
- Show or hide a password.
- Open or close a menu.
- Switch between light and dark mode.
- Expand or collapse an FAQ answer.
- Show or hide a pop-up.

Example:
```jsx
import { useState } from 'react';

function ThemeToggle() {
  const [darkMode, setDarkMode] = useState(false);

  return (
    <div>
      <p>{darkMode ? 'Dark mode is on' : 'Light mode is on'}</p>
      <button onClick={() => setDarkMode(!darkMode)}>
        Toggle Theme
      </button>
    </div>
  );
}
```

The `!darkMode` expression flips the boolean on every click: `false` becomes `true`, `true` becomes `false`. This is a very clean and readable pattern.

---

## 14. Counters

Counters are a classic beginner project because they are small but demonstrate the entire state-and-events loop.

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increase</button>
      <button onClick={() => setCount(count - 1)}>Decrease</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

What this teaches:
- How to initialize state.
- How to read state in JSX.
- How to update state from multiple different event handlers.
- How the UI responds automatically to every state change.

It is tiny, but it contains almost everything you need to understand before tackling more complex interactive UIs.

---

## 15. Show/Hide UI

Show/hide is just conditional rendering driven by boolean state:

```jsx
import { useState } from 'react';

function BioSection() {
  const [showBio, setShowBio] = useState(false);

  return (
    <div>
      <button onClick={() => setShowBio(!showBio)}>
        {showBio ? 'Hide Bio' : 'Show Bio'}
      </button>

      {showBio && (
        <p>Full stack developer and educator at Mctaba Labs.</p>
      )}
    </div>
  );
}
```

Notice how the button label also changes based on state. That is a very common and polished pattern. The same state drives both the button text and whether the bio paragraph appears.

This show/hide pattern is the foundation for:
- Modals and pop-ups.
- Dropdown menus.
- Accordion/FAQ components.
- Tab panels.
- Alert banners.

---

## 16. A Framework for Thinking About Interactive UI

Before writing any code, try this five-step process:

1. **Identify the visual states.** What does the UI look like in each possible situation? (e.g., password hidden vs. password visible)
2. **Decide what triggers state changes.** Is it a button click? A key press? A form submission?
3. **Represent that information in state.** Use the simplest type possible - booleans for on/off, numbers for counts, strings for text.
4. **Remove any state you do not actually need.** Less state means fewer bugs.
5. **Connect event handlers to update the state.** Wire up the event to the setter.

Applying this to a password toggle:
- Visual states: password hidden / password visible.
- Trigger: button click.
- State: `const [showPassword, setShowPassword] = useState(false)`.
- UI: `<input type={showPassword ? 'text' : 'password'} />`

By thinking through these steps before coding, you write cleaner components from the start.

---

## 17. Common Beginner Mistakes

| Mistake | Explanation |
|---|---|
| Using a regular variable instead of state | Normal variables do not persist between renders. React will not re-render when they change. |
| Directly assigning to the state variable (`count = count + 1`) | You must use the setter function. Direct assignment does not trigger a re-render. |
| Calling `onClick={handleClick()}` with parentheses | This runs the function immediately on render. Pass the function reference: `onClick={handleClick}`. |
| Placing `useState` inside a condition or loop | Hooks must be at the top level of the component, always called in the same order. |
| Forgetting to connect `value` and `onChange` on an input | A controlled input needs both. `value` without `onChange` makes the field read-only. |
| Expecting state to change immediately after calling the setter | State updates are asynchronous. The new value appears on the next render. |
| Confusing props and state | Props come from the parent and are read-only. State is managed inside the component and can change. |

---

## 18. Practice Projects

### A. Counter App
**Goal:** Build a counter with Increase, Decrease, and Reset buttons.  
**Concepts:** `useState`, click events, state updates, rendering based on state.  
**Stretch:** Prevent the counter from going below zero.

### B. Show/Hide Password Toggle
**Goal:** Build a password input with a button that reveals or hides the password text.  
**Concepts:** Boolean state, click events, conditional rendering, dynamic attributes.  
**Core pattern:** `<input type={showPassword ? 'text' : 'password'} />`

### C. FAQ Accordion
**Goal:** Build a list of question-and-answer items where clicking a question shows or hides the answer.  
**Concepts:** Toggles, conditional rendering, rendering lists, basic component structure.  
**Stretch:** Allow only one answer to be open at a time by managing state in the parent.

### D. Theme Toggle
**Goal:** Build a button that switches the page between light and dark mode.  
**Concepts:** Boolean state, conditional class names or text, click events.  
**Stretch:** Save the preference to `localStorage` so it persists after a refresh.

### E. Simple Registration Form
**Goal:** Build a form with live previewing as the user types.  
**Concepts:** Controlled inputs, multiple state values, `onChange`, conditional rendering.  
**Fields:** Name, email, password.  
**Stretch:** Disable the submit button until all fields have values. Show a success message after submission.

---

## 19. Checkpoint Questions

1. What is state in React, and how is it different from a regular JavaScript variable?
2. What does `useState` return?
3. Why must you use the setter function to update state instead of directly assigning to the variable?
4. What happens after you call a state setter function?
5. What is the difference between props and state?
6. How do you attach a click handler in JSX? What is the common mistake beginners make with this?
7. What does it mean for an input to be "controlled"?
8. Why is `onChange` necessary for a controlled input?
9. Name three ways to conditionally render content in JSX.
10. Why are counters and toggles good first React projects?

---

## 20. Assignment: Simple Registration Form

Build a **Registration Form** with the following requirements:

- Use `useState` for each input field: name, email, and password.
- Make all inputs controlled (each input has a `value` and an `onChange`).
- Display a live preview of the entered values below the form as the user types.
- Include a "Show/Hide Password" toggle button next to the password field.
- Disable the submit button if any field is empty.
- After the form is submitted, clear the fields and display a success message.

A strong submission demonstrates correct state management, clean event handling, and proper controlled inputs - not just a form that looks good on screen.
