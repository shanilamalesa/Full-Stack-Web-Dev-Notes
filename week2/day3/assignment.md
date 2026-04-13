# Week 2 - Day 3 Assignment

## Title
Flexbox and Grid -- Build a Responsive Photo Gallery

## Overview
You have suffered through layout with `float` and `position`. Today you use the modern tools. Flexbox is the one-dimensional layout system; Grid is the two-dimensional one. Your assignment is to build a responsive photo gallery using both: Flexbox for the navigation bar and card internals, Grid for the gallery layout itself.

## Learning Objectives Assessed
- Distinguish when Flexbox is the right tool (one dimension) vs Grid (two dimensions)
- Use `display: flex` with `justify-content`, `align-items`, `gap`, `flex-direction`
- Use `display: grid` with `grid-template-columns`, `gap`, and the `fr` unit
- Use `repeat(auto-fit, minmax(...))` to build a responsive grid without media queries

## Prerequisites
- Completed Day 1 and Day 2 assignments
- Your external stylesheet from Day 1 still linked

## AI Usage Rules

**Ratio this week:** 85% manual / 15% AI
**Habit:** Compare and contrast. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Writing one Flexbox or Grid rule with AI *after* you wrote your own, for comparison.
- **NOT ALLOWED FOR:** Generating the gallery CSS from scratch.
- **AUDIT REQUIRED:** Yes. Add a third Comparisons entry to `AI_AUDIT.md`.

## Tasks

### Task 1: Build a gallery page with 12 cards

**What to do:**
1. Create `gallery.html` in your portfolio repo.
2. Link to `main.css`.
3. Inside `<main>`, create a `<section class="gallery">` with 12 `<article class="gallery__card">` children. Each card should contain an `<img>`, an `<h3>` title, and a `<p>` caption.
4. Use placeholder images from `https://picsum.photos/seed/NAME/400/300` where NAME is different per card.

**Expected output:**
`gallery.html` with 12 cards in a single stack before you add any CSS.

### Task 2: Navigation bar with Flexbox

**What to do:**
Above the gallery, add a `<nav>` with your site name on the left, navigation links in the middle, and a "Contact" button on the right. Style it with Flexbox:

```css
nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 16px 24px;
}
```

The nav must stay horizontal at all screen sizes above 480px.

**Expected output:**
A clean horizontal nav with items aligned to left, centre, and right.

### Task 3: Gallery grid layout with `auto-fit` + `minmax`

**What to do:**
Style the `.gallery` container with CSS Grid:

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
  padding: 24px;
}
```

When you resize the browser, the number of columns should change automatically without you writing any media queries. Test by dragging the browser window from very narrow to very wide.

**Expected output:**
Gallery that shows 1 column on narrow screens, 2-3 on medium, 4+ on wide -- all without media queries.

### Task 4: Card internals with Flexbox

**What to do:**
Style each `.gallery__card` so its content is arranged with Flexbox in `flex-direction: column`. The image on top, the title below it, the caption at the bottom. Use `margin-top: auto` on the caption so it sticks to the bottom of the card regardless of title length.

**Expected output:**
All cards have the same visual height even if titles differ in length.

### Task 5: Compare with AI version

**What to do:**
Ask AI to rewrite your `.gallery` grid rule using a different approach (for example, with `grid-template-columns: repeat(3, 1fr)` plus media queries). Compare the two in `AI_AUDIT.md` and pick the winner. Which one actually responds to available space? Which one is shorter? Which one is clearer?

**Expected output:**
New entry in `AI_AUDIT.md` with the comparison and a defended final decision.

## Stretch Goals (Optional - Extra Credit)

- Add a hover effect that slightly zooms the image inside each card using `transform: scale(1.05)` and `transition`.
- Add a "Load More" button that does nothing yet (no JavaScript this week) but is visually styled.
- Add a dark-mode variant that swaps the card background and border colours.

## Submission Requirements

- **What to submit:** Portfolio repo link with `gallery.html`, updated `main.css`, and the new Comparisons entry in `AI_AUDIT.md`.
- **Where to submit:** Week 2 submissions channel
- **Deadline:** End of Day 3
- **Format:** Files inside existing portfolio repo

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Gallery grid responsive without media queries | 25 | `repeat(auto-fit, minmax(...))` correctly used. Resizing the browser changes column count without JS or breakpoints. |
| Flexbox nav correctly aligned | 15 | Site name left, links middle, button right. Stays horizontal on desktop. |
| Card internals with Flexbox column | 15 | Title and caption align properly. Card heights match even with uneven titles. |
| Correct choice of tool | 10 | Comment in CSS explaining why nav uses Flex and gallery uses Grid. |
| AI Audit entry | 15 | Comparisons entry with both versions, trade-offs, and defended pick. |
| Clean commit history | 10 | At least 2 commits with conventional messages. |
| Code quality | 10 | BEM-ish class names (`.gallery`, `.gallery__card`). No `!important`. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using Flexbox for everything.** Flexbox is great for one dimension. If you are fighting with vertical AND horizontal alignment at once, switch to Grid.
- **Fixed column counts.** `grid-template-columns: repeat(3, 1fr)` looks fine until a phone. Use `auto-fit` + `minmax` to get responsive behaviour for free.
- **Missing `gap`.** Do not use `margin` to space grid items -- `gap` is cleaner and handles edge cases automatically.

## Resources

- Day 1 reading: [CSS beyond the basics.md](../day1/CSS%20beyond%20the%20basics.md)
- Week 2 AI boundaries: [../ai.md](../ai.md)
- CSS Tricks "A Complete Guide to Flexbox": https://css-tricks.com/snippets/css/a-guide-to-flexbox/
- CSS Tricks "A Complete Guide to Grid": https://css-tricks.com/snippets/css/complete-guide-grid/
- Grid Garden game: https://cssgridgarden.com/
