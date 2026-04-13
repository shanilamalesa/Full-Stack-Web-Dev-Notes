# Week 2 - Day 1 Assignment

## Title
Style Your Week 1 Portfolio With External CSS, the Box Model, and Real Selectors

## Overview
Week 1 shipped a semantic HTML portfolio. This week you make it beautiful. Day 1's assignment extracts the "Portfolio Glow-Up" mini project from the Day 1 reading and expands it: you will create your first external stylesheet, wire it up to the portfolio pages you built last week, and use CSS selectors, the box model, colour, and typography to turn raw HTML into something that looks like a site. This is the first week you will compare a version you wrote by hand against a version AI generated for the same section.

## Learning Objectives Assessed
- Link an external stylesheet to multiple HTML pages
- Use element, class, and ID selectors correctly and understand when each is right
- Apply the CSS box model (margin, border, padding, content) with predictable results
- Write typography rules (font-family, line-height, letter-spacing, text-align)
- Use hex, rgb, and rgba colour values in a consistent palette

## Prerequisites
- Completed Week 1 assignments and weekend portfolio project
- Completed Day 1 readings: [introduction to css.md](./introduction%20to%20css.md) and [CSS beyond the basics.md](./CSS%20beyond%20the%20basics.md)
- Your Week 1 portfolio repository is still on GitHub

## AI Usage Rules

**Ratio this week:** 85% manual / 15% AI
**Habit:** Compare and contrast -- your version first, AI version second, judge both. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Asking AI to rewrite ONE section of your CSS *after* you have finished your own version, so you can compare them. Asking AI to explain a CSS property after you already read its MDN page.
- **NOT ALLOWED FOR:** Generating your stylesheet from scratch. Asking AI to "style my portfolio". Copying any AI-generated CSS into your submission without comparing it against your own first.
- **AUDIT REQUIRED:** Yes. Every week from Week 2 onwards requires an `AI_AUDIT.md` file in the repo. This week's audit must include a **Comparisons** section showing your version vs AI's version for at least one CSS block (see the template in [../ai.md](../ai.md)).

## Tasks

### Task 1: Create an external stylesheet and link it

**What to do:**
1. Inside your portfolio repository, create a folder `styles/` at the root.
2. Create a new file `styles/main.css`. Leave it empty for a moment.
3. In every HTML file in your portfolio (`index.html`, `bio.html`, `post.html`, and any others from the weekend project), add this line inside the `<head>`:

```html
<link rel="stylesheet" href="styles/main.css">
```

4. Adjust the relative path if your files are in a sub-folder.

**Expected output:**
Every page links to `styles/main.css`. You can verify by adding one test rule like `body { background: #f0f0f0; }` and confirming all pages show the grey background.

### Task 2: Define a colour palette and typography system

**What to do:**
At the very top of `main.css`, define a small palette in CSS custom properties. Write it by hand from scratch -- no AI, no copy-paste:

```css
:root {
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
  --color-accent: #2563eb;
  --color-muted: #6b7280;
  --font-body: 'Georgia', serif;
  --font-heading: 'Helvetica', sans-serif;
  --spacing-unit: 8px;
}
```

Then use the variables in at least 5 places across your stylesheet (for backgrounds, text colours, spacing).

Apply a consistent font-family and line-height to `body`. Pick your own values -- do not copy from AI.

**Expected output:**
Opening any page in the browser shows a consistent colour scheme and readable typography across all elements.

### Task 3: Use the box model on real elements

**What to do:**
Pick one element type from your portfolio (for example, `.post-card` or your `<article>` tags) and style it using **all four parts** of the box model:
- `margin` to push it away from other elements
- `border` with a colour and width
- `padding` to give the inner content space
- `width` or `max-width` to set the content size

Include a comment in your CSS above the rule explaining (in one line) what each property does. Example:

```css
/* margin = space outside the border; padding = space inside the border */
.post-card {
  max-width: 600px;
  margin: 24px auto;
  padding: 16px;
  border: 1px solid var(--color-muted);
}
```

**Expected output:**
The styled element is visibly spaced out and bounded on screen.

### Task 4: Use three different selector types in one stylesheet

**What to do:**
In `main.css`, include at least:
- One **element selector** (e.g., `h1 { ... }`)
- One **class selector** (e.g., `.tag { ... }`)
- One **ID selector** (e.g., `#hero { ... }`) -- but only one ID total in your stylesheet. IDs are for unique page landmarks.

For each, write a short comment explaining why you chose that selector type for that specific element. If you find yourself wanting to use more than one ID selector, rewrite it as a class.

**Expected output:**
All three selector types appear at least once in your CSS file with explanatory comments.

### Task 5: Run the Compare and Contrast exercise

**What to do:**
This is the new habit this week.

1. Pick one CSS block from your file that is at least 8 lines long (for example, the `.post-card` rule with margin/border/padding or your nav styling).
2. Open an AI tool of your choice. Share the corresponding HTML, then ask: "Here is my CSS for this element [paste]. Rewrite this as an alternative version. Do not explain -- just show me a different way to style the same element."
3. Save AI's version in a new file `styles/ai-comparison.css` (not linked to any page; this is for comparison only).
4. Create a file `AI_AUDIT.md` at the root of your repo and fill it out using the template in [../ai.md](../ai.md). Include:
   - The original prompt you used
   - Your version vs AI's version side by side (paste both)
   - At least 3 specific differences you noticed
   - Which version you are keeping (yours, theirs, or a hybrid) and **why**

**Expected output:**
`AI_AUDIT.md` committed at the repo root with a full Comparisons entry. `styles/ai-comparison.css` also committed for reference.

## Stretch Goals (Optional - Extra Credit)

- Add a `@media (prefers-color-scheme: dark)` rule that flips your palette to a dark theme using the same variables.
- Write one pseudo-class rule (`:hover`, `:focus`, or `:nth-child(2n)`) and explain in a comment what it does.
- Use the Prettier VS Code extension to format `main.css` automatically. Verify the diff is minimal and style-only.

## Submission Requirements

- **What to submit:**
  - Link to your updated portfolio repository (same repo from Week 1)
  - `styles/main.css` with the required sections
  - `styles/ai-comparison.css` with AI's alternative version
  - `AI_AUDIT.md` at the repo root with the Comparisons entry
- **Where to submit:** Week 2 submissions channel on Discord
- **Deadline:** End of Day 1, before Day 2 session
- **Format:** All files inside your existing portfolio repo

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| External stylesheet wired to all pages | 10 | `main.css` linked from every HTML file. Grey-background sanity check confirms it. |
| Colour palette and typography system | 15 | At least four custom properties defined at `:root` and used in at least 5 rules. Consistent fonts across pages. |
| Box model correctly applied | 15 | One element uses margin, border, padding, and width together. Comments explain each line. |
| Three selector types demonstrated | 15 | Element, class, and ID selectors each present at least once with explanatory comments. Only one ID total. |
| AI Audit Comparisons section | 25 | Prompt documented. Both versions pasted. At least 3 specific differences listed. Final decision defended in writing. A lazy audit ("AI was the same") loses the full 25 points. |
| Clean commit history | 10 | At least three commits for Day 1. Conventional commit prefixes (`feat:`, `refactor:`, `docs:`). |
| Code quality | 10 | Indentation consistent. Variables used instead of repeated literals. No `!important` in the file. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using IDs as style hooks.** IDs are unique and have the highest specificity of the three basic selectors. Use them for landmarks (`#main-nav`), not for general styling. Reach for a class instead.
- **Writing `margin` and `padding` twice.** Shorthand (`margin: 16px 24px`) can collide with specific sides (`margin-left: 8px`). Pick one style per rule.
- **Skipping the compare-and-contrast.** The audit is 25% of your grade this week. A half-done audit is worse than an incomplete assignment.

## Resources

- Day 1 reading: [introduction to css.md](./introduction%20to%20css.md)
- Day 1 reading: [CSS beyond the basics.md](./CSS%20beyond%20the%20basics.md)
- Week 2 AI boundaries: [../ai.md](../ai.md)
- MDN CSS Box Model: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model
- MDN CSS custom properties: https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties
