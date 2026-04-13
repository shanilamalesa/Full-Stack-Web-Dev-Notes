# Introduction to CSS

> **AI boundaries this week:** 85% manual / 15% AI. Habit: *Compare and contrast -- your version first, AI version second, judge both.* See [ai.md](../ai.md).

## Learning Objectives
By the end of this chapter, students will be able to:
- Understand what CSS is and its role in web development.
- Use inline, internal, and external CSS effectively.
- Target HTML elements using various selectors (Element, Class, ID).
- Apply basic styles: colors, typography, sizing, and spacing.
- Understand the **CSS Box Model** (Padding, Border, Margin).
- Build a professionally styled personal web page.

---

## 1. What is CSS?
**CSS (Cascading Style Sheets)** is the language used to style the visual presentation of web pages. While HTML provides the **structure** (the "skeleton"), CSS provides the **style** (the "skin, clothes, and makeup").

### The Web Layers
- **HTML**: Structure & Content (The "What")
- **CSS**: Presentation & Design (The "How it looks")
- **JavaScript**: Behavior & Interactivity (The "What it does")

CSS allows you to control colors, fonts, layouts, and even animations across multiple web pages simultaneously.

---

## 2. Ways to Apply CSS
There are three primary methods to add CSS to your HTML documents.

### 1. Inline CSS
Styles are added directly to an HTML element using the `style` attribute.
```html
<p style="color: red; font-weight: bold;">This is red and bold.</p>
```
- **Use Case**: Quick testing or styling a single, unique element.
- **Downside**: Hard to maintain; mixes content with presentation.

### 2. Internal (Embedded) CSS
Styles are defined within a `<style>` tag inside the `<head>` section of an HTML document.
```html
<head>
  <style>
    p {
      color: blue;
      line-height: 1.5;
    }
  </style>
</head>
```
- **Use Case**: Single-page websites or small projects.
- **Downside**: Styles cannot be reused on other pages.

### 3. External CSS (Best Practice)
Styles are written in a separate `.css` file and linked in the HTML `<head>`.
**File: `styles.css`**
```css
body {
  background-color: #f0f0f0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
```
**File: `index.html`**
```html
<head>
  <link rel="stylesheet" href="styles.css">
</head>
```
- **Use Case**: **Standard for professional development.**
- **Benefit**: Clean code, cached by browsers for faster loading, and reusable across the entire website.

---

## 3. CSS Syntax and Selectors
A CSS rule consists of a **selector** and a **declaration block**.

### The Anatomy of a CSS Rule
```css
h1 {
  color: teal;
  text-align: center;
}
```
- **Selector**: `h1` (Targets the element to style)
- **Property**: `color` (The feature you want to change)
- **Value**: `teal` (The setting for that property)

### Common Selectors
| Selector Type | Example | Targets |
| :--- | :--- | :--- |
| **Element** | `p { ... }` | All `<p>` elements on the page. |
| **Class** | `.btn { ... }` | All elements with `class="btn"`. (Reusable) |
| **ID** | `#header { ... }` | The single element with `id="header"`. (Unique) |
| **Universal** | `* { ... }` | Every single element on the page. |

---

## 4. The CSS Box Model
Every element in CSS is considered a rectangular box. Understanding the box model is key to controlling layout.

1. **Content**: The actual text or image.
2. **Padding**: Transparent space **inside** the border (between content and border).
3. **Border**: A line surrounding the padding and content.
4. **Margin**: Transparent space **outside** the border (between this element and others).

```css
.box {
  width: 300px;
  padding: 20px;
  border: 5px solid black;
  margin: 10px;
}
```

---

## 5. Essential Styling Properties

### Working with Colors
You can define colors in several ways:
- **Names**: `red`, `skyblue`, `transparent`.
- **HEX Codes**: `#ff5733` (Standard for web design).
- **RGB**: `rgb(255, 87, 51)` (Red, Green, Blue values 0-255).

### Typography
- `font-family`: Choose your font (e.g., `Arial, sans-serif`).
- `font-size`: Set text size (e.g., `16px`, `1.2rem`).
- `font-weight`: Make text `bold` or `normal`.
- `text-align`: Align text (`left`, `center`, `right`, `justify`).

---

## Mini Project: The Portfolio Glow-Up
Let's upgrade your `index.html` portfolio from the previous chapter.

### Step 1: Create `styles.css`
Create a new file and add these professional-looking styles:
```css
/* General Page Styling */
body {
  font-family: 'Helvetica Neue', Arial, sans-serif;
  background-color: #f8f9fa;
  color: #2d3436;
  line-height: 1.6;
  margin: 0;
  padding: 40px;
}

/* Headings */
h1, h2 {
  color: #0984e3;
  border-bottom: 2px solid #74b9ff;
  padding-bottom: 10px;
}

/* Navigation Links */
nav a {
  margin-right: 15px;
  text-decoration: none;
  color: #00b894;
  font-weight: bold;
}

nav a:hover {
  color: #00cec9;
}

/* Content Sections */
section {
  background: white;
  padding: 25px;
  margin-top: 25px;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
}

/* Footer */
footer {
  text-align: center;
  margin-top: 50px;
  font-size: 0.9rem;
  color: #636e72;
}
```

### Step 2: Link the Stylesheet
Ensure your `index.html` has this line in the `<head>`:
```html
<link rel="stylesheet" href="styles.css">
```

---

## Recap Quiz
1.  What are the three ways to apply CSS? Which is best for large projects?
2.  How do you target an element with the class `highlight`?
3.  What is the difference between `padding` and `margin`?
4.  Write a CSS rule to change the background color of all `<footer>` elements to light grey.
5.  What property would you use to change the font of your text?
