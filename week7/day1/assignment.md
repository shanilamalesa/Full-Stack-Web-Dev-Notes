# Week 7 - Day 1 Assignment

## Title
Create Your First React Project and Three Components By Hand

## Overview
Week 7 is the React week. Today you scaffold a fresh React project with Vite, delete the starter template, and build your first three components by hand. The "new framework, manual first" habit applies this week: the first 3-5 components you build get zero AI help so your brain encodes the JSX/props/render shape properly.

## Learning Objectives Assessed
- Set up a React project with Vite
- Write a function component that returns JSX
- Pass props from parent to child
- Compose multiple components in one page
- Explain what JSX compiles into

## Prerequisites
- Week 6 completed (async + fetch)
- Node installed and verified

## AI Usage Rules

**Ratio this week:** 60% manual / 40% AI
**Habit:** New framework, manual first -- the first 3-5 components by hand. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining JSX after you wrote your first component. Asking what a specific React warning means after you read the browser console.
- **NOT ALLOWED FOR:** Generating any of your first 5 components. Scaffolding React for you.
- **AUDIT REQUIRED:** Yes. React is a new framework, so include a "Manual-first components" log with the first 5 files, all hand-typed.

## Tasks

### Task 1: Create the project

**What to do:**
Open a terminal and run:

```bash
npm create vite@latest react-lab -- --template react
cd react-lab
npm install
npm run dev
```

Visit `http://localhost:5173`. The default Vite React starter page is running.

**Expected output:**
Starter page visible in the browser.

### Task 2: Delete the starter and write Hello World yourself

**What to do:**
1. Delete the content of `src/App.jsx`. Replace it with:

```jsx
function App() {
  return <h1>Hello, React</h1>;
}

export default App;
```

2. Delete the imports of CSS logos and the starter counter.
3. Save. The browser should reload showing "Hello, React".

Every character typed yourself. No copy-paste from the Vite starter.

**Expected output:**
Browser shows "Hello, React" with no starter artefacts visible.

### Task 3: Write three components by hand

**What to do:**
Create a folder `src/components/`. Inside, create three components as separate files:

1. `src/components/Greeting.jsx` -- takes a `name` prop and renders `<p>Hi, {name}!</p>`
2. `src/components/Avatar.jsx` -- takes a `src` and `alt` prop and renders an `<img>`
3. `src/components/ProfileCard.jsx` -- takes `name`, `role`, `avatar` props and renders a card using `Greeting` and `Avatar` together

Import and use `ProfileCard` in `App.jsx`:

```jsx
import ProfileCard from "./components/ProfileCard.jsx";

function App() {
  return (
    <div>
      <ProfileCard name="Wanjiru" role="Developer" avatar="https://placehold.co/80" />
      <ProfileCard name="Brian" role="Designer" avatar="https://placehold.co/80" />
    </div>
  );
}
```

Every component hand-typed. No AI.

**Expected output:**
Two profile cards visible in the browser.

### Task 4: Destructure props in the signature

**What to do:**
Refactor your three components so they destructure props in the function signature:

```jsx
// Before
function Greeting(props) {
  return <p>Hi, {props.name}!</p>;
}

// After
function Greeting({ name }) {
  return <p>Hi, {name}!</p>;
}
```

Apply this to all three components.

**Expected output:**
All three components use destructured props.

### Task 5: Understand what JSX compiles into

**What to do:**
In `jsx-notes.md`, write a short explanation (4-6 sentences) of:
- What JSX is (it looks like HTML but is not)
- What `<h1>Hello</h1>` compiles into (`React.createElement("h1", null, "Hello")`)
- Why React re-renders when you change state (preview of tomorrow)

Run this in a React file to confirm the compilation:

```jsx
import React from "react";
console.log(React.createElement("h1", null, "Hello from createElement"));
```

**Expected output:**
Notes committed. Console shows the React element object.

## Stretch Goals (Optional - Extra Credit)

- Render an array of profile objects with `.map` instead of hardcoding two cards.
- Add a `children` prop to `ProfileCard` and pass a custom footer into each.
- Use Fragments (`<>...</>`) to avoid wrapping in an extra `<div>`.

## Submission Requirements

- **What to submit:** Repo link with the new `react-lab` folder (or a separate `react-lab` repo), `jsx-notes.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Vite project scaffolded | 10 | Project runs on localhost with no errors. |
| Starter cleaned, Hello React by hand | 10 | No starter artefacts left. "Hello, React" visible. |
| Three components hand-built | 25 | Greeting, Avatar, ProfileCard all written by hand. Composition works. |
| Destructured props | 10 | All three components use destructuring. |
| JSX notes | 15 | Four questions answered in student's own words. createElement demo confirmed. |
| Manual-first audit | 20 | Audit shows the first 5 files were all hand-typed. If any AI-generated code slipped in, points lost. |
| Clean commits | 10 | Three-plus commits with conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Letting AI scaffold your components.** Week 7's habit is manual-first. The first 5 components must be yours. The audit will flag any AI-generated code in this range.
- **Missing file extensions in imports.** Vite prefers explicit `.jsx` extensions in imports. `import ProfileCard from "./components/ProfileCard.jsx"` not `...ProfileCard"`.
- **Returning two elements without a wrapper.** JSX requires a single root element. Use a Fragment `<>...</>` or a wrapping `<div>`.

## Resources

- Day 1 reading: [React Foundations.md](./React%20Foundations.md)
- Week 7 AI boundaries: [../ai.md](../ai.md)
- Vite guide: https://vitejs.dev/guide/
- react.dev "Learn" section: https://react.dev/learn
