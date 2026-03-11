# Forms in React

## Learning Goals

By the end of this topic, you should be able to:

- Build forms using **controlled inputs**
- Handle `onChange` and keep input values in React state
- Handle `onSubmit` correctly without a full page reload
- Validate inputs with basic rules (required, min length, email shape)
- Show validation error messages in the UI, near the relevant input
- Handle multiple fields cleanly using a single state object
- Reset the form state after successful submission
- Understand why libraries like **React Hook Form** exist and when to reach for them

---

## 1. Why Forms Matter

Forms are the backbone of real-world web products. Nearly every feature that collects user input is built on a form:

| Use Case | Example |
|---|---|
| Authentication | Login / Sign-up pages |
| Communication | Contact forms, support tickets |
| Commerce | Checkout and payment flows |
| Applications | Student or job application forms |
| User settings | Profile editing, preferences |
| Data exploration | Dashboards with filters and search |

If you can build good forms, you can build real applications. The React docs provide first-class support for HTML form elements through React DOM - components like `<form>`, `<input>`, and `<textarea>` all work in React with a few important differences from plain HTML.

---

## 2. Controlled Inputs - The React Way

### What is a controlled input?

In plain HTML, when a user types into an `<input>`, the browser keeps track of the value internally. React prefers to be *the single source of truth* for your UI, so instead, you store the input's current value in **React state** and tell React to update that state every time the user types.

This is called a **controlled input**: the value displayed always comes from state, and every keystroke updates that state through an event handler.

> **Key idea:** React drives the value. The browser never holds the value on its own.

### Example: one controlled input

```jsx
import { useState } from "react";

export default function ContactName() {
  // 1. Declare a state variable to hold the input's value
  const [name, setName] = useState("");

  return (
    <label>
      Name
      <input
        value={name}                          // 2. Bind value to state
        onChange={(e) => setName(e.target.value)} // 3. Update state on every keystroke
        placeholder="Your name"
      />
    </label>
  );
}
```

**Step-by-step walkthrough:**

1. `useState("")` creates a `name` variable that starts as an empty string, and a `setName` function to update it.
2. `value={name}` tells React: *always show whatever is in `name`*. The input no longer manages its own value.
3. `onChange` fires every time the user types a character. `e.target.value` is the full current string in the input. We pass it to `setName`, React re-renders, and the input shows the new value.

### Why controlled inputs are useful

| Benefit | Why it matters |
|---|---|
| React always knows the current value | You can read it anytime without querying the DOM |
| Validation is straightforward | Check state values at any point |
| Live previews are easy | Display `{name}` anywhere on the page instantly |
| Clean resets | Set state to `""` and the input clears |
| Conditional UI | Disable a submit button if fields are empty |

---

## 3. The `onChange` Event

`onChange` is the event React uses to respond to the user typing into an input. Every time the user presses a key, `onChange` fires and gives you a synthetic event object `e`. From that event, `e.target.value` is the current full string in the input field.

### Pattern to memorize

```jsx
<input value={stateValue} onChange={(e) => setStateValue(e.target.value)} />
```

This three-part pattern - **state value → bind to `value` → update in `onChange`** - is the foundation of every controlled form in React. You'll write this hundreds of times, so it should become second nature.

> **Note:** Unlike HTML's `oninput`, React's `onChange` behaves more like `oninput` - it fires on every keystroke, not just when the field loses focus. This is intentional and makes real-time validation possible.

---

## 4. `onSubmit` and Form Submission

### The problem with HTML form submission

In plain HTML, clicking a submit button causes the browser to reload the page and send form data to a URL. In a React single-page app, you almost never want that - you want to handle the data yourself (validate it, send it to an API, show a success message, etc.).

### Solution: `e.preventDefault()` + your own logic

React's `<form>` element supports an `onSubmit` prop. When the form is submitted, you call `e.preventDefault()` to stop the browser reload, then run your own logic.

### Example: basic form submit

```jsx
import { useState } from "react";

export default function ContactForm() {
  const [email, setEmail] = useState("");

  function handleSubmit(e) {
    e.preventDefault(); // Stop the browser from reloading the page

    // At this point you have full control - validate, call an API, or log
    console.log("Submitting email:", email);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Email
        <input
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="you@example.com"
        />
      </label>

      <button type="submit">Send</button>
    </form>
  );
}
```

### `onSubmit` vs `onClick`

| | `onSubmit` on `<form>` | `onClick` on `<button>` |
|---|---|---|
| Triggered by pressing **Enter** in a field | Yes | No |
| Triggered by clicking the submit button | Yes | Yes |
| Standard form UX | Yes | Not quite |

**Always use `onSubmit` on the `<form>` element** - it handles both mouse clicks and keyboard submission, which is what users expect.

---

## 5. Handling Multiple Fields

Real forms have multiple fields. There are two common patterns for managing their state.

### Approach A: One `useState` per field

Good for simple forms with just two or three fields.

```jsx
const [name, setName] = useState("");
const [email, setEmail] = useState("");
const [password, setPassword] = useState("");

// Each field gets its own handler:
<input value={name} onChange={(e) => setName(e.target.value)} />
<input value={email} onChange={(e) => setEmail(e.target.value)} />
```

**Downside:** As the form grows, this gets repetitive and hard to manage. Resetting means setting every variable separately.

---

### Approach B: One object state for all fields(recommended)

Group all field values into a single state object. One change handler works for every field, as long as each `<input>` has a `name` attribute matching the object key.

```jsx
import { useState } from "react";

export default function SignupForm() {
  const [form, setForm] = useState({
    name: "",
    email: "",
    password: "",
  });

  function handleChange(e) {
    const { name, value } = e.target;
    // Spread the previous state, then overwrite just the changed field
    setForm((prev) => ({ ...prev, [name]: value }));
  }

  function handleSubmit(e) {
    e.preventDefault();
    console.log("Form data:", form);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"          // "name" matches form.name
        value={form.name}
        onChange={handleChange}
        placeholder="Name"
      />

      <input
        name="email"         // "email" matches form.email
        value={form.email}
        onChange={handleChange}
        placeholder="Email"
      />

      <input
        name="password"      // "password" matches form.password
        value={form.password}
        onChange={handleChange}
        placeholder="Password"
        type="password"
      />

      <button type="submit">Create account</button>
    </form>
  );
}
```

**How `handleChange` works:**

```js
function handleChange(e) {
  const { name, value } = e.target;
  // e.target.name  → the "name" attribute of the input that changed (e.g. "email")
  // e.target.value → what the user typed
  setForm((prev) => ({ ...prev, [name]: value }));
  // [name] is a computed property key - it dynamically uses the name attribute
  // ...prev keeps all other fields unchanged
}
```

**Why this pattern is powerful:**

- One handler for every field - no duplication
- Adding a new field only requires adding it to the state object and writing one `<input>`
- Easy to reset: `setForm({ name: "", email: "", password: "" })`
- Easy to submit: `form` is already a ready-to-send data object

---

## 6. Form Validation Basics

**Validation** means checking that the user's input meets your requirements *before* you accept or submit it. This protects your application from bad data and gives users clear feedback.

### Common validation rules for beginners

| Rule | Example |
|---|---|
| Required field | Name cannot be empty |
| Minimum length | Password must be at least 8 characters |
| Email format | Must contain `@` and a domain |
| Passwords match | Confirm password equals password |
| Checkbox accepted | User must agree to terms |

React does not have a built-in validation system. Instead, you write a validation function that receives your form state and returns an object of error messages - one per failing field. This is idiomatic React because it uses state and conditional rendering, which you already know.

### A reusable validation function

```jsx
function validate(form) {
  const errors = {}; // Start with an empty errors object

  // Name validation
  if (!form.name.trim()) {
    errors.name = "Name is required.";
  }

  // Email validation
  if (!form.email.trim()) {
    errors.email = "Email is required.";
  } else if (!form.email.includes("@")) {
    // Only check format if the field isn't empty
    errors.email = "Enter a valid email address.";
  }

  // Password validation
  if (!form.password) {
    errors.password = "Password is required.";
  } else if (form.password.length < 8) {
    errors.password = "Password must be at least 8 characters.";
  }

  return errors;
  // If errors is an empty object {}, all validations passed
  // If it has any keys, those fields failed
}
```

**How to use it:**

```jsx
const nextErrors = validate(form);

if (Object.keys(nextErrors).length === 0) {
  // No errors - safe to submit
} else {
  // Show errors to the user
  setErrors(nextErrors);
}
```

---

## 7. Showing Validation Messages

Validation only helps if users can see the errors. Best practice: display each error message *directly below the input it refers to*. Use conditional rendering (`&&`) to show the error only when it exists.

### Full example: signup form with validation

```jsx
import { useState } from "react";

export default function SignupForm() {
  const [form, setForm] = useState({ name: "", email: "", password: "" });
  const [errors, setErrors] = useState({});
  const [success, setSuccess] = useState(false);

  function handleChange(e) {
    const { name, value } = e.target;
    setForm((prev) => ({ ...prev, [name]: value }));
  }

  function handleSubmit(e) {
    e.preventDefault();

    const nextErrors = validate(form);
    setErrors(nextErrors);

    if (Object.keys(nextErrors).length === 0) {
      // All validations passed - submit the form
      console.log("Submitting:", form);
      setSuccess(true);
      setForm({ name: "", email: "", password: "" }); // reset fields
      setErrors({});                                   // clear errors
    }
  }

  if (success) {
    return <p>Account created successfully! Welcome aboard.</p>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name
        <input name="name" value={form.name} onChange={handleChange} />
        {/* Show the error only if errors.name exists */}
        {errors.name && <p style={{ color: "red" }}>{errors.name}</p>}
      </label>

      <label>
        Email
        <input name="email" value={form.email} onChange={handleChange} />
        {errors.email && <p style={{ color: "red" }}>{errors.email}</p>}
      </label>

      <label>
        Password
        <input
          type="password"
          name="password"
          value={form.password}
          onChange={handleChange}
        />
        {errors.password && <p style={{ color: "red" }}>{errors.password}</p>}
      </label>

      <button type="submit">Sign up</button>
    </form>
  );
}

function validate(form) {
  const errors = {};
  if (!form.name.trim()) errors.name = "Name is required.";
  if (!form.email.trim()) errors.email = "Email is required.";
  else if (!form.email.includes("@")) errors.email = "Enter a valid email address.";
  if (!form.password) errors.password = "Password is required.";
  else if (form.password.length < 8) errors.password = "Password must be at least 8 characters.";
  return errors;
}
```

### What's happening with `{errors.name && <p>...</p>}`?

This is **short-circuit evaluation** - a JavaScript trick used for conditional rendering in JSX:

- If `errors.name` is `undefined` (no error), the whole expression evaluates to `undefined` and React renders nothing.
- If `errors.name` is a string like `"Name is required."`, the `&&` returns the right side, and React renders the `<p>` tag.

---

## 8. Resetting Form State

After a successful submission, a good form resets itself. This means:

1. **Clear all field values** - so new entries don't conflict with old ones
2. **Clear all errors** - stale errors shouldn't hang around
3. **Show a success message** - let the user know it worked

### Reset pattern

```jsx
// Reset all fields to their initial empty values
setForm({ name: "", email: "", password: "" });

// Clear all validation errors
setErrors({});

// Optionally show a success state
setSuccess(true);
```

### Success message using conditional rendering

```jsx
const [success, setSuccess] = useState(false);

// In your JSX:
if (success) {
  return (
    <div>
      <p>Your message was sent. We'll be in touch!</p>
      <button onClick={() => setSuccess(false)}>Send another</button>
    </div>
  );
}

// Otherwise render the form...
```

> **Tip:** Return early from the component when `success` is `true`. This completely swaps the form for a success view - clean and readable.

---

## 9. The Complete Submission Flow

Every professional form follows this mental model. Memorize it:

```
1. User fills in the form fields
       ↓
2. User clicks Submit (or presses Enter)
       ↓
3. onSubmit fires → e.preventDefault() stops page reload
       ↓
4. Run validate(form)
       ↓
5. Errors found?
   ├── YES → setErrors(nextErrors) → show errors → STOP
   └── NO  → proceed ↓
       ↓
6. Send data (console.log, API call, etc.)
       ↓
7. Show success message
       ↓
8. Reset form state
```

Every real-world form - login, checkout, application, settings - follows this exact flow.

---

## 10. Common Beginner Mistakes

Understanding what *not* to do is just as important as knowing the right pattern.

| Mistake | Why it's a problem | Fix |
|---|---|---|
| Forgetting `e.preventDefault()` | Page reloads on submit, losing all state | Always add it as the first line of `handleSubmit` |
| Not adding `value` prop to input | React can't control the input - it becomes "uncontrolled" | Always connect `value={stateValue}` |
| Forgetting `name` attribute in object-state approach | `e.target.name` will be `undefined`, breaking `handleChange` | Every input needs a `name` that matches the state key |
| Never clearing errors | After fixing a mistake, the old error message stays visible | Reset `errors` on success; clear individual field errors `onChange` if needed |
| Storing redundant state | e.g., storing both `errors` and `isValid` | `isValid` can always be *derived* from `errors` with `Object.keys(errors).length === 0` |
| Showing errors at the top of the form only | Users have to scroll up to read which field failed | Show each error message directly below its field |

---

## 11. Later: React Hook Form

Once you can build forms from scratch with controlled inputs, you'll encounter **React Hook Form** - a popular library that dramatically reduces boilerplate.

### Why learn the manual way first?

Because React Hook Form *abstracts* controlled inputs, state management, and validation. If you don't understand those concepts individually, you won't know how to debug problems or customize behavior when something goes wrong.

Think of it like learning arithmetic before using a calculator.

### What React Hook Form adds

| Feature | Benefit |
|---|---|
| `useForm()` hook | Replaces all your `useState` for form fields |
| `register()` | Connects inputs without writing `value` and `onChange` manually |
| `handleSubmit()` | Wraps your submit logic and runs validation automatically |
| `formState.errors` | Structured errors from validation rules |
| Re-render optimization | Only the changed input re-renders - much faster on large forms |

**A taste of the syntax (don't worry about this yet):**

```jsx
import { useForm } from "react-hook-form";

export default function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  function onSubmit(data) {
    console.log(data); // { email: "...", password: "..." }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("email", { required: "Email is required" })} />
      {errors.email && <p>{errors.email.message}</p>}

      <input type="password" {...register("password", { minLength: { value: 8, message: "Min 8 chars" } })} />
      {errors.password && <p>{errors.password.message}</p>}

      <button type="submit">Login</button>
    </form>
  );
}
```

Once your fundamentals are solid, this library will feel like a superpower.

---

## 12. Later: Zod Schema Validation

As your applications grow more complex, you'll want a more structured way to define and enforce validation rules. **Zod** is a TypeScript-first schema validation library that lets you describe exactly what shape your data should take.

### What Zod lets you do

```js
import { z } from "zod";

const signupSchema = z.object({
  name: z.string().min(1, "Name is required"),
  email: z.string().email("Enter a valid email"),
  password: z.string().min(8, "Password must be at least 8 characters"),
});

// Validate data against the schema:
const result = signupSchema.safeParse(form);

if (!result.success) {
  console.log(result.error.flatten().fieldErrors);
  // { name: ["Name is required"], email: ["Enter a valid email"], ... }
}
```

### When Zod becomes useful

- Your forms have many complex rules that are hard to manage in a single `validate()` function
- You want to **share validation logic** between your frontend form and a backend API
- You're working with TypeScript and want type-safe data shapes
- You're using React Hook Form - Zod pairs with it via a "resolver"

> **Note:** Zod is not part of React itself - it's a separate library. Master plain React form validation first, then layer Zod on top.

---

## 13. Practice Projects

Build these projects in order of complexity. Each one reinforces the concepts from this topic.

---

### A) Contact Form

**Fields:**
- Name
- Email
- Message (use `<textarea>`)

**Requirements:**
- All fields are required
- Validate email format (must contain `@`)
- Show error messages below each field
- Show a success message after valid submission
- Reset all fields after success

> `<textarea>` works the same as `<input>` in React - give it `value` and `onChange`:
> ```jsx
> <textarea value={form.message} onChange={handleChange} name="message" />
> ```

---

### B) Student Application Form (McTaba Labs-style)

**Fields:**
- Full name
- Email
- Phone number
- Experience level (`<select>` dropdown)
- "Why do you want to join?" (`<textarea>`)

**Requirements:**
- All fields required (phone can be a simple non-empty check)
- Experience level must be selected (not just the placeholder option)
- Show "Thanks for applying, we'll be in touch!" after valid submit
- Reset form after success

> For a `<select>`, the pattern is the same:
> ```jsx
> <select name="experience" value={form.experience} onChange={handleChange}>
>   <option value="">-- Select level --</option>
>   <option value="beginner">Beginner</option>
>   <option value="intermediate">Intermediate</option>
>   <option value="advanced">Advanced</option>
> </select>
> ```

---

### C) Login / Signup Form

Build **both** forms, optionally with a toggle between them.

**Login fields:**
- Email
- Password

**Signup fields:**
- Name
- Email
- Password
- Confirm password

**Requirements:**
- Email must be a valid format
- Password must be at least 8 characters
- Confirm password must match password exactly
- Stretch goal: disable the submit button until all fields pass validation

---

### D) Checkout Form UI (Frontend only)

**Fields:**
- Full name
- Phone number
- Delivery address
- Payment method (radio buttons: M-Pesa / Card / Cash on delivery)
- Order notes (optional `<textarea>`)

**Requirements:**
- Validate all required fields (notes is optional - skip validation for it)
- Show a static "Order Summary" section beside or below the form
- Show a "Order confirmed!" success message after valid submission

> For radio buttons:
> ```jsx
> <label>
>   <input
>     type="radio"
>     name="payment"
>     value="mpesa"
>     checked={form.payment === "mpesa"}
>     onChange={handleChange}
>   />
>   M-Pesa
> </label>
> ```

---

## 14. End-of-Topic Summary

| Concept | Key takeaway |
|---|---|
| Controlled inputs | `value` comes from state; `onChange` updates state on every keystroke |
| `onSubmit` | Attach to `<form>`, not the button. Always call `e.preventDefault()` first |
| Multiple fields | Use a single object in state + a shared `handleChange` using `e.target.name` |
| Validation | Write a `validate(form)` function that returns an `errors` object |
| Error display | Show errors below each input using conditional rendering (`{errors.field && <p>...}`) |
| Reset | Call `setForm(initialState)` and `setErrors({})` after success |
| React Hook Form | A library to reduce boilerplate - learn it after mastering the manual approach |
| Zod | Schema validation - useful for complex rules and shared backend/frontend validation |

---

## Official React Documentation References

- [`<form>` element reference](https://react.dev/reference/react-dom/components/form)
- [`<input>` element reference](https://react.dev/reference/react-dom/components/input)
- [`<textarea>` element reference](https://react.dev/reference/react-dom/components/textarea)
- [`<select>` element reference](https://react.dev/reference/react-dom/components/select)
- [Conditional rendering](https://react.dev/learn/conditional-rendering)
- [Choosing the state structure](https://react.dev/learn/choosing-the-state-structure)
- [Reacting to input with state](https://react.dev/learn/reacting-to-input-with-state)
