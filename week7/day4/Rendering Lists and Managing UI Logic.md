# Rendering Lists and Managing UI Logic

## Learning Objectives
By the end of this lesson, you will be able to:
- Render arrays of data as lists of JSX elements using `.map()`.
- Use the `key` prop correctly and understand why it matters.
- Filter displayed data using `.filter()` before rendering.
- Handle common conditional UI states: loading, empty, and results.
- Build a basic live search input connected to filtered list rendering.
- Structure UI logic cleanly using `if`, `&&`, and ternary operators.

---

## 1. Why This Topic Matters

Most interesting React interfaces are not static pages with hardcoded content. They are **data-driven** - they read from a list, filter that list, and display a result. A student directory, task list, product grid, movie browser, or notifications panel all depend on rendering many items from an array.

The mindset shift this lesson teaches is: do not write the same markup over and over by hand. Store your data in an array, and let React generate the interface from that data. When the data changes, the UI updates automatically.

This is one of the most important transitions from "building static pages" to "building real applications."

---

## 2. Rendering Arrays with `.map()`

The primary tool for rendering lists in React is JavaScript's `.map()` method. You call it on an array of data, and it transforms each item into a JSX element. React then renders all of those elements on screen.

```jsx
const students = [
  { id: 1, name: 'Amina' },
  { id: 2, name: 'Brian' },
  { id: 3, name: 'Linet' }
];

function StudentList() {
  return (
    <ul>
      {students.map((student) => (
        <li key={student.id}>{student.name}</li>
      ))}
    </ul>
  );
}
```

What is happening here, step by step:
1. `students` is an array of objects.
2. `.map()` loops through each object in the array.
3. For each object, we return a `<li>` element containing that student's name.
4. React takes the resulting array of `<li>` elements and renders them all inside the `<ul>`.

This is the same as writing out each `<li>` by hand, except that you only write the template once, and it works for any number of items in the array.

---

## 3. Why `.map()` Is the Right Tool

You might wonder: why not use a `for` loop? The reason is that `.map()` is an **expression** - it returns a value (an array of JSX elements) that can be placed directly inside your JSX. A `for` loop is a **statement** - it executes code but does not return a value.

JSX only works with expressions inside `{}` curly braces, so `.map()` is the natural fit.

You will use this pattern constantly in React for:
- Student directories
- Product grids
- Comment sections
- Testimonials
- Task lists
- Blog post feeds
- Notifications

---

## 4. The `key` Prop

Whenever you render a list with `.map()`, React requires a `key` prop on each element. Without it, React will show a warning in the console.

```jsx
{students.map((student) => (
  <li key={student.id}>{student.name}</li>
))}
```

### Why Keys Matter

React needs to efficiently update the DOM when a list changes - for example, when items are added, removed, sorted, or filtered. Keys are how React identifies which item is which between renders.

Without keys, React has to guess which DOM elements correspond to which list items. This guessing can cause bugs: wrong items get re-rendered, component state gets attached to the wrong position, or transitions animate the wrong elements.

With a stable, unique key, React can track each item precisely regardless of how the list changes.

### What Makes a Good Key

A good key is **unique** and **stable** - it does not change between renders.

Use:
- A database `id` (best option when available).
- A slug or URL-safe name if guaranteed unique.

Avoid:
- The array index (e.g., `key={index}`). Indices change when items are sorted, filtered, or reordered, which defeats the purpose of having a key.
- Randomly generated values (e.g., `Math.random()`). These change every render, causing React to destroy and recreate every list item every time.

The simplest rule: if your data has an `id` field, use it as the key.

---

## 5. Filtering Lists with `.filter()`

`.filter()` creates a new array containing only the items that pass a given condition. You use it when you want to show a subset of the data - for example, only active students, only in-stock products, or only tasks that are not completed.

```jsx
const students = [
  { id: 1, name: 'Amina', active: true },
  { id: 2, name: 'Brian', active: false },
  { id: 3, name: 'Linet', active: true }
];

function ActiveStudents() {
  const activeStudents = students.filter((student) => student.active);

  return (
    <ul>
      {activeStudents.map((student) => (
        <li key={student.id}>{student.name}</li>
      ))}
    </ul>
  );
}
```

The pattern is:
1. Call `.filter()` to narrow the array to only the items you want.
2. Call `.map()` on the filtered result to turn those items into JSX.

You can chain them in a single expression if you prefer:

```jsx
const visibleStudents = students
  .filter((student) => student.active)
  .map((student) => (
    <li key={student.id}>{student.name}</li>
  ));
```

This `filter + map` combination is one of the most common patterns in React. You will see it in almost every data-driven interface.

---

## 6. Conditional Rendering

Not every situation calls for showing the same UI. The UI often needs to change depending on conditions: whether data is loading, whether there are results, whether a user is logged in. React handles all of this using plain JavaScript - there is no special React syntax for it.

### Using `if`

Use `if` when you need to return completely different JSX from a component:

```jsx
function StudentList({ students }) {
  if (students.length === 0) {
    return <p>No students found.</p>;
  }

  return (
    <ul>
      {students.map((student) => (
        <li key={student.id}>{student.name}</li>
      ))}
    </ul>
  );
}
```

This is the cleanest option when you need to bail out of a component early and show something entirely different.

### Using the Ternary Operator

Use a ternary when you need to choose between two pieces of JSX inline:

```jsx
{students.length === 0 ? (
  <p>No students found.</p>
) : (
  <ul>
    {students.map((student) => (
      <li key={student.id}>{student.name}</li>
    ))}
  </ul>
)}
```

The ternary format is `condition ? valueIfTrue : valueIfFalse`. It works well inside JSX when you need to pick between two outcomes without interrupting the flow of the markup.

### Using `&&`

Use `&&` when you want to show something only when a condition is true, and show nothing at all when it is false:

```jsx
{isNew && <span>New</span>}
```

If `isNew` is `true`, the `<span>` renders. If `isNew` is `false`, nothing is shown.

> One warning: avoid using `&&` with numbers where the number could be zero. `{0 && <Component />}` will render the number `0` on screen rather than nothing, because 0 is a falsy value but not `false`. Use a boolean conversion or a ternary in that case.

---

## 7. Empty States

An **empty state** is the UI you show when there is nothing to display. A blank section with no explanation looks like a bug. A clear message tells the user exactly what is happening.

```jsx
function StudentDirectory({ students }) {
  if (students.length === 0) {
    return <p>No students found.</p>;
  }

  return (
    <ul>
      {students.map((student) => (
        <li key={student.id}>{student.name}</li>
      ))}
    </ul>
  );
}
```

Good empty state messages are specific:
- "No tasks yet. Add one above."
- "No products match your search."
- "Your watchlist is empty."
- "No students found."

Always build an empty state. It is a small addition that makes the app feel complete and intentional.

---

## 8. Loading States

A **loading state** is the placeholder UI you show while data is being prepared or fetched. Before the data arrives, the user should see something that signals the app is working.

```jsx
function MovieList({ isLoading, movies }) {
  if (isLoading) {
    return <p>Loading movies, please wait...</p>;
  }

  return (
    <ul>
      {movies.map((movie) => (
        <li key={movie.id}>{movie.title}</li>
      ))}
    </ul>
  );
}
```

Common loading messages:
- "Loading..."
- "Fetching products..."
- "Searching..."

Even before you learn about fetching data from an API, understand the concept of a loading state. It is part of good UI logic. An interface that appears to do nothing while work is happening feels broken - a loading message removes that ambiguity.

---

## 9. Building a Search Input

A search input is one of the most practical examples of connecting state, controlled inputs, filtering, and conditional rendering all at once. It is the pattern behind nearly every list-based interface.

```jsx
import { useState } from 'react';

const students = [
  { id: 1, name: 'Amina' },
  { id: 2, name: 'Brian' },
  { id: 3, name: 'Linet' }
];

function StudentSearch() {
  const [query, setQuery] = useState('');

  const filteredStudents = students.filter((student) =>
    student.name.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search students"
      />

      {filteredStudents.length === 0 ? (
        <p>No students match your search.</p>
      ) : (
        <ul>
          {filteredStudents.map((student) => (
            <li key={student.id}>{student.name}</li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

What is happening here:
1. `query` stores the current search text. It starts as an empty string.
2. Each keystroke updates `query` through the `onChange` handler.
3. `filteredStudents` is derived: we call `.filter()` on the full list using the current `query` value. This calculation runs on every render.
4. If `filteredStudents` is empty, show an empty state message.
5. Otherwise, render the filtered results with `.map()`.

Note that `filteredStudents` is not stored in state. It is **computed during render** from two values that are already available: the full student list and the current query. This is cleaner than storing a separate filtered list in state, because storing derived data in state means you have to keep two values in sync manually.

---

## 10. Deriving Data vs Storing Extra State

A key principle for clean list-based UI: if a value can be **calculated** from data you already have, do not store it in state.

Consider this comparison:

**Unnecessary extra state:**
```jsx
const [products, setProducts] = useState(allProducts);
const [filteredProducts, setFilteredProducts] = useState(allProducts); // duplicate!
const [query, setQuery] = useState('');
```

**Derived on render (cleaner):**
```jsx
const [query, setQuery] = useState('');

const filteredProducts = allProducts.filter((product) =>
  product.title.toLowerCase().includes(query.toLowerCase())
);
```

In the second version, `filteredProducts` is recalculated on every render based on `query`. You only maintain one piece of state (`query`), and the filtered list always stays in sync automatically.

The rule: if you can compute a value from existing state or props, do not store it as separate state. Compute it instead.

---

## 11. Extracting List Items into Components

When a list item becomes more complex than one line of text, extract it into its own component. This keeps your list component focused on rendering logic and your item component focused on display.

```jsx
function StudentCard({ student }) {
  return (
    <div className="student-card">
      <h3>{student.name}</h3>
      <p>{student.track}</p>
      <p>{student.location}</p>
    </div>
  );
}

function StudentDirectory({ students }) {
  return (
    <section>
      {students.map((student) => (
        <StudentCard key={student.id} student={student} />
      ))}
    </section>
  );
}
```

The `key` goes on the outermost element returned by `.map()` - in this case, on the `<StudentCard />` component itself, not inside the `StudentCard` definition.

---

## 12. A Mental Checklist for List-Based UI

When building any list-based screen, ask yourself:

1. What array holds the data I want to display?
2. Do I need to filter or search it before rendering?
3. What unique field should I use as the `key` for each item?
4. What should the UI show when the list is empty?
5. What should the UI show while data is loading?
6. Should the list item be a reusable component, or is it simple enough to keep inline?
7. Can any values I need be derived from existing state, or do I need to store new state?

Working through this checklist before writing code will help you plan cleaner, more complete interfaces.

---

## 13. Common Beginner Mistakes

| Mistake | Why It Is a Problem | What to Do Instead |
|---|---|---|
| Forgetting the `key` prop | React cannot track items; list updates may produce bugs | Use a stable unique id from your data |
| Using the array index as the key | Indices shift when items are filtered or sorted, causing wrong behavior | Use a real id |
| Manually writing repeated markup by hand | Does not scale; hard to update | Store data in an array and use `.map()` |
| Storing a filtered list in separate state | Creates out-of-sync state; harder to maintain | Derive it during render with `.filter()` |
| No empty state | Users see a blank screen and assume something is broken | Always add an empty state message |
| Putting all display logic in one giant component | Hard to read and maintain | Extract item display into a separate component |

---

## 14. Practice Projects

### A. Student Directory
**Goal:** Display a list of students from an array, with a search bar.  
**Concepts:** `.map()`, keys, controlled input, `.filter()`, empty state.  
**Data:** Each student has `id`, `name`, `track`, `location`, `status`.  
**Stretch:** Add an Active/Inactive filter toggle alongside the search.

### B. Task List
**Goal:** Render a list of tasks with visual distinction for completed items.  
**Concepts:** `.map()`, keys, conditional rendering, `.filter()`.  
**Data:** Each task has `id`, `title`, `completed`.  
**Stretch:** Add "All", "Completed", and "Pending" filter tabs.

### C. Product List with Search
**Goal:** Build a product grid with a live search input.  
**Concepts:** Controlled input, `.filter()` + `.map()`, keys, empty state, conditional badges.  
**Data:** Each product has `id`, `title`, `price`, `category`, `inStock`.  
**Stretch:** Add an "In Stock Only" checkbox that filters alongside the search text.

---

## 15. Checkpoint Questions

1. Why do React apps use `.map()` for rendering lists instead of manually writing each item?
2. What is the purpose of the `key` prop? What happens if you leave it out?
3. Why is using the array index as a `key` often a bad idea?
4. How do `.filter()` and `.map()` work together in a typical React list?
5. What is an empty state, and why should you always include one?
6. What is a loading state, and when would you use it?
7. What is the difference between the `&&` operator and the ternary operator for conditional rendering? When would you use each?
8. Why is it better to derive filtered results from existing state rather than storing a separate filtered array?
9. When should you extract a list item into its own component?
10. Looking at the search example in section 9 of these notes: why is `filteredStudents` not stored in state?

---

## 16. Assignment: Product List with Search

Build a **Product List with Search** that meets the following requirements:

- Store at least 8 products in an array. Each product should have: `id`, `title`, `price`, `inStock`.
- Render the products using `.map()` with a proper `key` on each item.
- Create a reusable `ProductCard` component for displaying a single product.
- Add a controlled search input that filters the list by product title as the user types.
- Show a clear empty state message when no products match the search.
- Conditionally show an "In Stock" or "Out of Stock" label on each card based on the `inStock` value.
- Add an "In Stock Only" checkbox that filters out unavailable products when checked.

A strong submission combines a clean search filter, a proper empty state, correct list rendering with stable keys, and clear conditional rendering logic - all organized into focused, reusable components.
