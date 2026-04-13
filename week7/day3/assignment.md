# Week 7 - Day 3 Assignment

## Title
State, useState, and Event Handlers -- Build a Counter and a Toggle

## Overview
Today state arrives. `useState` is the first hook you will use, and it is the core of how React makes UIs feel alive. Your assignment is two classic first-React-projects -- a Counter and a Toggle -- both hand-written. Then you extend them with AI help once the manual versions work.

## Learning Objectives Assessed
- Use `useState` to create and update local state
- Handle `onClick` events that update state
- Understand that calling a setter triggers a re-render
- Manage controlled boolean state for toggles
- Refactor repeated state logic into reusable patterns

## Prerequisites
- Day 1 and Day 2 completed

## AI Usage Rules

**Ratio:** 60/40. **Habit:** Build first, enhance with AI. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Extending a working counter with AI after your manual version works.
- **NOT ALLOWED FOR:** Writing your first `useState` call for you.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Build a Counter component by hand

**What to do:**
Create `src/components/Counter.jsx`:

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
      <button onClick={() => setCount(0)}>reset</button>
    </div>
  );
}

export default Counter;
```

Type it yourself. Use it in `App.jsx`.

**Expected output:**
Counter visible in the browser. Clicks update the number.

### Task 2: Build a Toggle component by hand

**What to do:**
Create `src/components/Toggle.jsx`:

```jsx
import { useState } from "react";

function Toggle() {
  const [isOn, setIsOn] = useState(false);

  return (
    <button onClick={() => setIsOn(!isOn)}>
      {isOn ? "ON" : "OFF"}
    </button>
  );
}

export default Toggle;
```

Use it in `App.jsx`.

**Expected output:**
Toggle button that switches between ON and OFF.

### Task 3: Counter with functional state updates

**What to do:**
Add a "double" button to your Counter. Naive implementation:

```jsx
<button onClick={() => {
  setCount(count + 1);
  setCount(count + 1);
}}>+2</button>
```

This will only increment by 1. Why? Because both `setCount` calls see the same stale `count`. Fix with functional updates:

```jsx
<button onClick={() => {
  setCount((c) => c + 1);
  setCount((c) => c + 1);
}}>+2</button>
```

In `day3-notes.md`, explain the difference in 3-4 sentences.

**Expected output:**
Counter with working +2 button. Notes explain the fix.

### Task 4: Extract a custom toggle into a function

**What to do:**
You wrote the toggle inline. Extract it into a named function:

```jsx
function Toggle() {
  const [isOn, setIsOn] = useState(false);

  function handleToggle() {
    setIsOn((prev) => !prev);
  }

  return (
    <button onClick={handleToggle}>
      {isOn ? "ON" : "OFF"}
    </button>
  );
}
```

Named handlers are the preferred pattern when the logic gets non-trivial. Using a named function here is practice for Day 4.

**Expected output:**
Toggle with named handler.

### Task 5: Extend with AI

**What to do:**
After your Counter works, ask AI to extend it with a feature: "Add a step input that lets me set how much each click increments by. Here is my working counter [paste]. Only change what is necessary."

Add the extension. Record in `AI_AUDIT.md`:
- Your working manual version (link to commit)
- Your prompt
- AI's version
- What you kept
- What you changed

**Expected output:**
Counter with a step input. Audit entry complete.

## Stretch Goals (Optional - Extra Credit)

- Add a "history" state that records the last 10 values and displays them.
- Add `localStorage` persistence -- the counter remembers its value across page reloads (hint: you will need `useEffect`, which is next week).
- Add keyboard handlers so pressing `+` and `-` on the keyboard updates the count.

## Submission Requirements

- **What to submit:** Repo with updated `react-lab`, `day3-notes.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Counter component | 20 | Works with +, -, reset. All by hand. |
| Toggle component | 15 | Works with ON/OFF. |
| Functional state update fix | 20 | +2 button works correctly. Notes explain why the naive version fails. |
| Named handler in Toggle | 10 | Refactor done. |
| AI extension of counter | 20 | Step input added via build-first audit. |
| Clean commits | 10 | Conventional messages. |
| Code quality | 5 | Descriptive names, consistent style. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Mutating state directly.** `count = count + 1` does nothing. You must call the setter. React only re-renders when the setter is called.
- **Calling the setter with a stale value in quick succession.** Always use functional updates `setCount((c) => c + 1)` when the new value depends on the previous one.
- **Using `onclick` instead of `onClick`.** React uses camelCase for all event handlers. HTML `onclick` won't work in JSX.

## Resources

- Day 3 reading: [State and Events.md](./State%20and%20Events.md)
- Week 7 AI boundaries: [../ai.md](../ai.md)
- react.dev useState: https://react.dev/reference/react/useState
