# Components Deep Dive

## Learning Objectives
By the end of this lesson, you will be able to:
- Explain what props are and how they work.
- Pass data from a parent component to a child component.
- Render components dynamically using different data.
- Build reusable components instead of copying and pasting markup.
- Use the `children` prop to build flexible wrapper components.
- Understand component composition.
- Explain the difference between **props** and **state**.
- Keep components small, focused, and predictable.

---

## 1. Why This Topic Matters

In the Foundations lesson, you built static components like `Navbar`, `Hero`, or `ProfileCard`. They always showed the same content. That is a good start, but it is not how real React apps work.

Real React development begins when you build components that accept **different inputs** and produce **different outputs** based on that input. This is what **props** unlock.

Think about a list of testimonials on a website. Instead of writing a separate testimonial card for every person, you build one `TestimonialCard` component and pass in different data for each one. That is the power of props. One component, many uses.

---

## 2. Props

### What Are Props?

**Props** (short for "properties") are the data you pass into a component from its parent. Think of a component like a function: props are the arguments you pass in, and the JSX that comes out is the result.

```
props = inputs to a component
component = function  
JSX returned = output
```

### A Basic Example

Without destructuring (the older style):
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

Used like this:
```jsx
<Welcome name="Bonaventure" />
```

### Destructuring Props (The Modern Style)

Most React developers use **destructuring** to unpack props directly in the function signature. It is shorter and easier to read:

```jsx
function Welcome({ name }) {
  return <h1>Hello, {name}</h1>;
}
```

Both produce the same result. Prefer the destructured version in new code.

### Why Props Matter

Without props, a component can only ever show one fixed thing. With props, the same component can show many different versions of itself based on the data it receives. That is what makes React components powerful and reusable.

---

## 3. Passing Data from Parent to Child

A **parent component** renders a **child component** and passes values to it through JSX attributes - just like attributes on an HTML tag.

```jsx
function TeamMember({ name, role }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{role}</p>
    </div>
  );
}

export default function App() {
  return (
    <div>
      <TeamMember name="Amina" role="Frontend Developer" />
      <TeamMember name="Kevin" role="Backend Developer" />
    </div>
  );
}
```

In this example:
- `App` is the **parent** component.
- `TeamMember` is the **child** component.
- `name` and `role` are **props** being passed from parent to child.

### The Golden Rule: Data Flows Downward

In React, data flows in one direction: **from parent to child**. A child component receives props from its parent. It cannot push data back up the tree on its own. This makes the app easier to reason about because you always know where data is coming from.

---

## 4. Dynamic Rendering

**Dynamic rendering** means the UI changes based on the props it receives. Instead of hardcoding content inside a component, you let the incoming props decide what gets displayed.

```jsx
function ProductCard({ title, price }) {
  return (
    <div>
      <h3>{title}</h3>
      <p>KES {price}</p>
    </div>
  );
}
```

You can now render completely different cards just by passing in different values:

```jsx
<ProductCard title="Wireless Mouse" price={1200} />
<ProductCard title="Mechanical Keyboard" price={4500} />
<ProductCard title="USB-C Hub" price={2500} />
```

This is **data-driven UI**. The component knows nothing about specific products - it just knows how to display whatever data it is given.

---

## 5. Reusability

Reusability means you write a component once and use it in many places with different data. It is one of the main reasons developers choose React.

### The Problem: Repeating Markup

```jsx
<div className="testimonial">
  <h3>Sarah</h3>
  <p>This bootcamp changed my career.</p>
</div>

<div className="testimonial">
  <h3>David</h3>
  <p>I finally understand how to build real projects.</p>
</div>
```

This works, but if you want to change the design of the card you have to update every single copy manually. As the list grows, this becomes a maintenance nightmare.

### The Solution: A Reusable Component

```jsx
function TestimonialCard({ name, quote }) {
  return (
    <div className="testimonial">
      <h3>{name}</h3>
      <p>{quote}</p>
    </div>
  );
}
```

Now you can render as many testimonials as you need by just passing in different data:

```jsx
<TestimonialCard name="Sarah" quote="This bootcamp changed my career." />
<TestimonialCard name="David" quote="I finally understand how to build real projects." />
```

If the design ever changes, you update one component and every card updates everywhere automatically.

### The Instinct to Build

Whenever you see repeated UI, ask yourself:
- Is this the same structure, just with different data?
- Should this become a reusable component?
- Can I pass the differences in as props?

Build that habit early. It is the heart of React thinking.

---

## 6. Types of Values You Can Pass as Props

Props are not limited to strings. You can pass in any valid JavaScript value.

| Prop Type | Example |
|---|---|
| String | `<ProfileCard name="Bonaventure" />` |
| Number | `<ProductCard price={2500} />` |
| Boolean | `<Badge isNew={true} />` |
| Object | `<UserCard user={{ name: "Amina", role: "Designer" }} />` |
| Array | `<Tags tags={["React", "JavaScript"]} />` |
| Function | `<Button onClick={handleClick} />` |
| JSX | `<Layout left={<Sidebar />} />` |

Note: non-string values (numbers, booleans, objects, arrays, functions) must be passed inside curly braces `{}`.

---

## 7. Default Values for Props

If a parent does not pass a certain prop, the component will receive `undefined` for that value. You can set a **default value** directly in the destructuring to handle this gracefully:

```jsx
function Avatar({ name, size = 80 }) {
  return (
    <div>
      <p>{name}</p>
      <p>Size: {size}px</p>
    </div>
  );
}
```

If you render `<Avatar name="Mary" />` without passing `size`, the component will automatically use `80`. This makes components flexible without requiring the parent to always provide every single prop.

---

## 8. The `children` Prop

### What Is `children`?

When you wrap content between the opening and closing tags of a component, that content is automatically available inside the component as a special prop called `children`.

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>;
}
```

Used like this:

```jsx
<Card>
  <h2>Welcome</h2>
  <p>This content is inside the card.</p>
</Card>
```

Everything between `<Card>` and `</Card>` gets passed in as `children` and rendered wherever `{children}` appears in the component.

### Why Is This Useful?

The `children` pattern lets you build **wrapper components** - components that provide common styling or structure while letting the parent decide exactly what goes inside them.

Without `children`:
- You would need a different Card component for every type of content.

With `children`:
- One Card component works for any content: headings, paragraphs, buttons, images, other components - anything.

### Common Uses for `children`

- `Card`
- `Modal`
- `Layout`
- `Section`
- `Container`
- `ButtonGroup`

All of these are wrapper-style components. They give the outside world a structural shell; the parent fills in the contents.

---

## 9. Component Composition

**Component composition** is the practice of building larger UI pieces by combining smaller, focused components together.

Instead of building one giant component that does everything, you build small components that each do one thing well, then combine them:

```jsx
function Avatar({ image, name }) {
  return <img src={image} alt={name} />;
}

function TeamMemberInfo({ name, role }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>{role}</p>
    </div>
  );
}

function TeamMemberCard({ image, name, role }) {
  return (
    <div className="team-member-card">
      <Avatar image={image} name={name} />
      <TeamMemberInfo name={name} role={role} />
    </div>
  );
}
```

Here, `TeamMemberCard` is **composed of** `Avatar` and `TeamMemberInfo`. Each sub-component has one clear responsibility. The larger card becomes much easier to understand, update, and reuse.

### Why Composition Beats Giant Components

A component that tries to handle everything in one block becomes:
- Hard to read (you have to scroll a lot).
- Hard to reuse (it is too specific).
- Hard to update (changing one thing can affect everything else).
- Hard to debug.

Small, composed components are the opposite: easy to understand, easy to swap out, and easy to test.

---

## 10. Rendering Lists of Components

Once you understand reusable components, the next natural step is rendering them from an array of data. The JavaScript `map()` method is used to loop over an array and turn each item into a JSX element.

```jsx
const teamMembers = [
  { id: 1, name: "Amina", role: "Frontend Developer" },
  { id: 2, name: "Brian", role: "Backend Developer" },
  { id: 3, name: "Lilian", role: "UI Designer" }
];

function TeamMember({ name, role }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>{role}</p>
    </div>
  );
}

export default function TeamSection() {
  return (
    <section>
      {teamMembers.map((member) => (
        <TeamMember
          key={member.id}
          name={member.name}
          role={member.role}
        />
      ))}
    </section>
  );
}
```

### The `key` Prop

Notice the `key={member.id}` above. When you render a list of components, React needs a unique `key` on each one so it can track which item is which when the list changes. Always pass a unique, stable value (like an ID from your data) as the key. Avoid using the array index as a key if the list can be reordered or filtered.

### Why This Pattern Is Powerful

This pattern appears in almost every real-world React app:
- Product listings
- Blog posts
- Testimonials
- Comments
- Notifications
- Search results

Instead of hardcoding 10 product cards, you store the product data in an array and let `map()` generate the cards for you. If the data changes, the UI updates automatically.

---

## 11. Conditional Rendering with Props

Props can also control **whether** content appears at all, not just what the content says.

### Using the `&&` Operator

If a condition is `true`, render the element. If it is `false`, render nothing:

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

If `isNew` is `true`, the `<strong>New</strong>` tag appears. If `isNew` is `false`, nothing is rendered.

### Using a Ternary

To choose between two pieces of content:

```jsx
function TestimonialCard({ name, quote, featured }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>{quote}</p>
      {featured ? <span>Featured</span> : null}
    </div>
  );
}
```

Conditional rendering makes the same component work in more situations, without having to create different versions of it for each case.

---

## 12. Props vs. State

This is one of the most important distinctions in React. Beginners often confuse these two concepts.

| | Props | State |
|---|---|---|
| Where it comes from | Passed in by the parent | Managed inside the component |
| Who controls it | The parent | The component itself |
| Can the component change it? | No (read-only) | Yes |
| Used for | Receiving configuration and data | Storing values that change over time |

### A Simple Mental Model

- **Props = external input.** Someone gives you a value; you use it.
- **State = internal memory.** You store and update a value yourself.

### A Practical Guide

At this stage, use props for everything. If a value comes from a parent, it is a prop. You will learn about `useState` in the next lesson, which introduces state for interactivity (like a button click counter or a toggle that shows/hides a panel).

React's "Thinking in React" guide specifically says: build the static version with props first, before adding any state. State is only for things that change over time in response to user interaction or fetched data.

---

## 13. Props Are Read-Only

A child component must never modify the props it receives. Props are given to you to read and use - not to edit.

Bad approach:

```jsx
function ProductCard(props) {
  props.price = 0; // Never do this!
  return <p>{props.price}</p>;
}
```

Modifying props creates unpredictable bugs that are very hard to trace. Other parts of the app depend on that data being stable.

Correct approach:

```jsx
function ProductCard({ price }) {
  return <p>{price}</p>;
}
```

If a component needs to track a changing value internally, that is what **state** is for, introduced in the next lesson.

---

## 14. Passing JSX as a Named Prop

You have seen how `children` works for nesting content between a component's tags. React also supports passing JSX as a named prop directly. This is useful when a component has more than one "content slot."

```jsx
function Layout({ left, right }) {
  return (
    <div className="layout">
      <aside>{left}</aside>
      <main>{right}</main>
    </div>
  );
}
```

Used like this:

```jsx
<Layout
  left={<Sidebar />}
  right={<Content />}
/>
```

The `Layout` component does not know or care what `Sidebar` or `Content` are. It just knows where to put them. This is a flexible and clean composition pattern.

---

## 15. Common Beginner Mistakes

| Mistake | What to Do Instead |
|---|---|
| Hardcoding everything instead of using props | Build one reusable component and pass data in as props |
| Trying to modify props inside the child | Treat props as read-only; use state for values that change |
| Confusing props and state | Props come from outside; state is managed inside the component |
| Building one giant component | Break large UIs into smaller, focused components |
| Forgetting about `children` | Use `children` for wrapper components that hold other content |
| Manually writing repeated markup | Use `map()` to generate lists of components from data arrays |

---

## 16. Best Practices

1. **Start static, then add data.** Build the component with hardcoded values first to make sure the structure is right, then switch to props.
2. **Use descriptive prop names.** Props like `name`, `price`, `imageUrl`, `isNew`, and `onSubmit` communicate intent clearly.
3. **Destructure your props.** It is the cleaner, modern style and makes the component signature self-documenting.
4. **Keep components focused.** If a component is doing too many different things, split it up.
5. **Use `map()` for lists.** Never manually write the same JSX multiple times for repeated data.

---

## 17. Practice Projects

### A. Team Members Section
**Goal:** Build a section displaying multiple team members using a reusable component.  
**Concepts practiced:** Props, parent-to-child data flow, list rendering, composition.  
**Component structure:** `TeamSection` > `TeamMemberCard` > `Avatar` + `TeamMemberInfo`  
**Stretch:** Add a `featured` boolean prop and conditionally render a "Featured" badge.

### B. Testimonials Section
**Goal:** Build a testimonials section for a landing page.  
**Concepts practiced:** Reusable components, props, list rendering, conditional rendering.  
**Component structure:** `TestimonialsSection` > `TestimonialCard`  
**Stretch:** Wrap each testimonial in a reusable `Card` component using `children`.

### C. Product Card Grid
**Goal:** Build a grid of product cards for a simple e-commerce layout.  
**Concepts practiced:** Props, array rendering with `map()`, conditional UI, children, composition.  
**Component structure:** `ProductGrid` > `ProductCard` > `PriceTag` + `Badge` + `Button`  
**Stretch:** Use a wrapper `Card` component with `children` to hold the card layout.

---

## 18. Checkpoint Questions

Use these to test your understanding before moving on:

1. What are props in React, and why are they useful?
2. How does a parent component pass data to a child component?
3. What is the difference between props and state?
4. What does the `children` prop represent? Give an example of when you would use it.
5. Why should you never modify a prop inside a child component?
6. What is component composition and why is it better than one giant component?
7. How does `map()` help when building repeated component UI?
8. What is the purpose of the `key` prop when rendering lists?
9. What kinds of values can be passed as props?
10. When should repeated markup be extracted into a reusable component?

---

## 19. Assignment: Team Members Section

Build a **Team Members Section** with the following requirements:

- Create a reusable `TeamMemberCard` component.
- Pass all data from a parent component into the card using props.
- Render at least 4 team members from an array using `map()`.
- Include at least one nested component inside the card (for example, a separate `Avatar` component for the image).
- Use one wrapper component that accepts `children`.
- Keep components in separate files inside a `components/` folder.
- Do not manually repeat the same JSX more than once.

A strong submission demonstrates clean component thinking and correct parent-to-child data flow, not just getting something to display on screen.
