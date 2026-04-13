# Week 2 - Day 2 Assignment

## Title
Layout Fundamentals -- Normal Flow, Display Types, and Positioning

## Overview
Yesterday you styled individual elements. Today you arrange them on the page. You will build a small multi-component layout (header, nav, main, sidebar, footer) using only `display`, `position`, and the normal document flow -- no Flexbox or Grid yet. This deliberate restriction teaches you what the modern tools are actually solving for.

## Learning Objectives Assessed
- Explain how normal document flow stacks elements
- Use `display: block`, `inline`, `inline-block` appropriately
- Apply `position: static`, `relative`, `absolute`, `fixed`, `sticky` correctly
- Understand containing blocks and when `position: absolute` breaks out of flow
- Use `z-index` only when necessary and justify it

## Prerequisites
- Completed Day 1 assignment (external stylesheet, palette, box model)
- Read the layout sections of [CSS beyond the basics.md](../day1/CSS%20beyond%20the%20basics.md)
- `main.css` from Day 1 still active and linked

## AI Usage Rules

**Ratio this week:** 85% manual / 15% AI
**Habit:** Compare and contrast. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Asking AI to rewrite one layout block after you finish your version. Asking AI to explain a positioning quirk after you reproduced it yourself.
- **NOT ALLOWED FOR:** Generating the full layout. Asking AI "how should I lay out my page?".
- **AUDIT REQUIRED:** Yes. Add one new Comparisons entry to your existing `AI_AUDIT.md` from Day 1.

## Tasks

### Task 1: Build a 5-region layout with plain CSS

**What to do:**
Create a new file `layout-lab.html` in your portfolio repo with this structure:

```html
<body>
  <header>Site Header</header>
  <nav>Top Nav</nav>
  <main>
    <article>Main Article</article>
    <aside>Sidebar</aside>
  </main>
  <footer>Site Footer</footer>
</body>
```

In `main.css` (or a new `styles/layout.css` if you prefer), style these regions using ONLY `display` and `position` -- NO Flexbox, NO Grid.

Requirements:
- `header` is full-width and 80px tall
- `nav` is sticky to the top when scrolling past it
- `main > article` takes the left 70% of the available width
- `main > aside` takes the right 30% using `float` or `display: inline-block`
- `footer` is full-width at the bottom

**Expected output:**
A rendered page that shows all five regions in their correct positions on screen.

### Task 2: Demonstrate each `position` value once

**What to do:**
Add five small elements somewhere on `layout-lab.html`, each demonstrating one of the positioning values. Label each one visually with its position name:

```html
<div class="demo demo--static">static (default)</div>
<div class="demo demo--relative">relative (nudged 20px right)</div>
<div class="demo demo--absolute">absolute (pinned top-right of parent)</div>
<div class="demo demo--fixed">fixed (stays visible on scroll)</div>
<div class="demo demo--sticky">sticky (sticks after scroll)</div>
```

Style each one using the appropriate `position` value. Next to each CSS rule, include a comment explaining what the positioning does and what it breaks out of (if anything).

**Expected output:**
All five positioned elements visible on the page, each behaving differently when you scroll.

### Task 3: Explain containing blocks in writing

**What to do:**
Create a short file `layout-notes.md` in your repo. In 6-8 sentences, explain:
1. What "normal flow" means
2. What a "containing block" is
3. Why `position: absolute` uses the nearest positioned ancestor as its containing block (and what happens if there is none)

Use your own words. No AI. Paste the file into your audit if you quoted from anywhere.

**Expected output:**
`layout-notes.md` committed at the repo root.

### Task 4: Compare your layout with an AI version

**What to do:**
Pick your `main > article` + `aside` layout block. Ask AI to rewrite it as an alternative approach. Save it in a commented-out block at the bottom of your stylesheet. Add a new Comparisons entry to `AI_AUDIT.md` covering:
- Your approach (float or inline-block?)
- AI's approach
- Trade-offs between them
- Which one you shipped and why

**Expected output:**
Updated `AI_AUDIT.md` with a second Comparisons entry.

## Stretch Goals (Optional - Extra Credit)

- Add a back-to-top button using `position: fixed` at the bottom-right of the viewport.
- Create a "modal" using `position: fixed` with a semi-transparent `rgba()` backdrop.
- Add one `z-index` rule and write a comment explaining why you needed it.

## Submission Requirements

- **What to submit:** Portfolio repo link with `layout-lab.html`, updated `main.css` (or `layout.css`), `layout-notes.md`, and updated `AI_AUDIT.md`.
- **Where to submit:** Week 2 submissions channel
- **Deadline:** End of Day 2
- **Format:** Files inside existing portfolio repo

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| 5-region layout works | 20 | Header, nav, main, article, aside, footer all positioned correctly. Page renders without broken regions. |
| Sticky nav works on scroll | 10 | Nav becomes sticky when scrolled past. |
| Five positioning demos | 15 | Each position value (static, relative, absolute, fixed, sticky) visible and behaving correctly. |
| layout-notes.md quality | 15 | All three questions answered in student's own words. No copy-paste. |
| AI Audit Comparisons | 20 | New entry with prompt, both versions, trade-offs, and final decision. |
| Clean commit history | 10 | At least 2 commits with conventional messages. |
| Code quality | 10 | Comments on each positioning rule. No Flexbox or Grid used (this week only). |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using Flexbox or Grid this week.** You will meet them on Day 3 and 4. The point of today is to feel the old pain so you appreciate the new tools.
- **Forgetting that `position: absolute` needs a positioned ancestor.** If nothing above your element has `position: relative` (or absolute/fixed), the containing block falls back to the viewport. Add `position: relative` to the parent that should contain it.
- **Overusing `z-index`.** If you find yourself writing `z-index: 9999`, stop. Fix the stacking context instead.

## Resources

- Day 1 reading: [CSS beyond the basics.md](../day1/CSS%20beyond%20the%20basics.md) (sections on layout and positioning)
- Week 2 AI boundaries: [../ai.md](../ai.md)
- MDN containing block: https://developer.mozilla.org/en-US/docs/Web/CSS/Containing_block
- MDN position property: https://developer.mozilla.org/en-US/docs/Web/CSS/position
