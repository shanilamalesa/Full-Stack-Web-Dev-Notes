# Week 7 - Day 2 Assignment

## Title
Components Deep Dive -- Props, Children, and Composition

## Overview
Day 1 you wrote your first components. Today you master the props model: passing functions as props, using the `children` prop, lifting shared UI into reusable components, and composing them intentionally. Still no state -- that is tomorrow.

## Learning Objectives Assessed
- Pass any value as a prop (string, number, function, object, array)
- Use the `children` prop to build layout components
- Compose small components into larger ones
- Build reusable UI primitives (Button, Card, Container)

## Prerequisites
- Day 1 completed

## AI Usage Rules

**Ratio:** 60/40. **Habit:** New framework, manual first -- still in the first 3-5 hand-typed components range. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining `children` after you built your first layout component.
- **NOT ALLOWED FOR:** Generating your `Card`, `Button`, or `Container` primitives.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Three UI primitives

**What to do:**
In `src/components/`, create three reusable primitives:

1. `Button.jsx` -- takes `children` and `onClick`, renders a styled `<button>`. Ignore the click handler for now (we do state tomorrow).
2. `Card.jsx` -- takes `title` and `children`, renders a `<div>` with a title and a body.
3. `Container.jsx` -- takes `children`, renders a `<div>` with max-width and centered margin.

```jsx
// Button.jsx
function Button({ children, onClick }) {
  return (
    <button onClick={onClick} style={{ padding: "8px 16px" }}>
      {children}
    </button>
  );
}

export default Button;
```

**Expected output:**
Three primitive components created, exported, importable.

### Task 2: Compose a simple page

**What to do:**
In `App.jsx`, use your primitives to build a page with two Cards inside a Container:

```jsx
<Container>
  <Card title="Welcome">
    <p>This is the first card.</p>
    <Button>Click me</Button>
  </Card>
  <Card title="About">
    <p>This is the second card.</p>
    <Button>More info</Button>
  </Card>
</Container>
```

**Expected output:**
Two cards rendered inside a centered container. Both have buttons.

### Task 3: Pass functions as props

**What to do:**
Instead of hardcoding the button click behaviour inside `Button`, pass the handler as a prop:

```jsx
<Button onClick={() => alert("Card 1 clicked")}>Click me</Button>
```

Verify the click actually fires the alert. (This uses the onClick you wired in Task 1.)

**Expected output:**
Alert appears when button is clicked.

### Task 4: Render a list of cards

**What to do:**
Hardcode an array at the top of `App.jsx`:

```jsx
const cards = [
  { id: 1, title: "Home", body: "Welcome home" },
  { id: 2, title: "About", body: "About us" },
  { id: 3, title: "Contact", body: "Get in touch" },
];
```

Render the array as cards using `.map`:

```jsx
<Container>
  {cards.map((card) => (
    <Card key={card.id} title={card.title}>
      <p>{card.body}</p>
    </Card>
  ))}
</Container>
```

Notice the `key` prop. React warns if you forget it.

**Expected output:**
Three cards rendered from the array, no console warning about keys.

### Task 5: Explain the key prop

**What to do:**
In `key-notes.md`, write 4-6 sentences answering:
- Why does React need a `key` on list items?
- What happens when you use the array index as a key?
- What is a good source of a key? (hint: unique, stable, from the data)

Your own words. Read the React docs first.

**Expected output:**
Notes committed.

## Stretch Goals (Optional - Extra Credit)

- Add a `variant` prop to `Button` that accepts "primary" or "secondary" and applies different styles.
- Build a `Grid` component that accepts `children` and renders them in a CSS Grid layout.
- Pass a `renderFooter` function prop to `Card` so the parent can customise the card's footer.

## Submission Requirements

- **What to submit:** Repo with updated `react-lab`, `key-notes.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Three UI primitives | 20 | Button, Card, Container all working and exported. |
| Composed page with two cards | 15 | Page renders correctly via Container + Card + Button. |
| Function passed as prop | 15 | Button onClick fires the parent handler. |
| List rendering with .map | 20 | Three cards from array, key prop correct, no console warnings. |
| key prop notes | 15 | Three questions answered in student's own words. |
| Manual-first audit | 10 | Day 2 files still hand-typed. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using array index as key.** It works until you reorder or insert. Use stable IDs from your data instead.
- **Returning multiple elements without a wrapper.** Same rule as Day 1 -- use Fragment or a single parent.
- **Mutating props.** Props are read-only. Never write to `props.name = ...` -- React does not track it and your UI will be out of sync with reality.

## Resources

- Day 2 reading: [Components Deep Dive.md](./Components%20Deep%20Dive.md)
- Week 7 AI boundaries: [../ai.md](../ai.md)
- React docs: Passing props to a component: https://react.dev/learn/passing-props-to-a-component
- React docs: Rendering lists: https://react.dev/learn/rendering-lists
