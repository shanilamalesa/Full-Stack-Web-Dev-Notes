# Week 2 - Day 4 Assignment

## Title
Responsive Design, Media Queries, and the Week 2 Ship Checklist

## Overview
Day 4 is pre-weekend polish day. You make every page of your portfolio responsive across three breakpoints (mobile, tablet, desktop), add CSS transitions for hover states, and run through a completion checklist that proves the portfolio is ready for Monday. By the end of today the portfolio you built in Week 1 should be visually polished, responsive, and deployable.

## Learning Objectives Assessed
- Write mobile-first media queries at three standard breakpoints
- Use `min-width` queries instead of `max-width` for mobile-first workflow
- Apply CSS transitions to hover states
- Complete a ship-ready checklist verifying visual consistency across pages

## Prerequisites
- Completed Day 1, 2, 3 assignments
- All portfolio pages link to `main.css`

## AI Usage Rules

**Ratio this week:** 85% manual / 15% AI
**Habit:** Compare and contrast. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Asking AI to suggest reasonable breakpoint values after you looked at your actual content breaking.
- **NOT ALLOWED FOR:** Generating all your media queries. Designing breakpoints without looking at your own content first.
- **AUDIT REQUIRED:** Yes. Final Comparisons entry for Week 2 must be in the audit.

## Tasks

### Task 1: Find your natural breakpoints

**What to do:**
1. Open each page in your browser and open DevTools.
2. Use the device toolbar to resize the viewport from 320px wide up to 1400px wide.
3. Note the widths at which the layout looks bad. Write them down on paper.
4. Pick three breakpoints: a mobile one (typically around 640px), a tablet one (around 768px or 1024px), and a desktop one (around 1280px).

**Expected output:**
A short note in `layout-notes.md` listing the three breakpoints and one sentence explaining why you picked each.

### Task 2: Mobile-first media queries

**What to do:**
In `main.css`, refactor your existing styles so the base rules look right at 320px (the smallest mobile). Then add three `@media (min-width: ...)` blocks that add desktop adjustments as the screen grows:

```css
/* Base: mobile styles, no media query */
.card { padding: 12px; }

/* Tablet */
@media (min-width: 768px) {
  .card { padding: 20px; }
}

/* Desktop */
@media (min-width: 1280px) {
  .card { padding: 24px; max-width: 800px; }
}
```

Every media query must be `min-width` (mobile-first). Do not mix `max-width` queries.

**Expected output:**
Resizing the browser shows at least three distinct visual states across your portfolio pages.

### Task 3: Transitions on interactive elements

**What to do:**
Add a subtle hover transition on at least three interactive elements: links, buttons, and cards. The transition should feel smooth (not instant, not slow):

```css
.btn {
  transition: background-color 200ms ease, transform 200ms ease;
}
.btn:hover {
  background-color: var(--color-accent);
  transform: translateY(-2px);
}
```

**Expected output:**
Hovering over links, buttons, and cards triggers a smooth visual change.

### Task 4: Week 2 Ship Checklist

**What to do:**
Create or update `CHECKLIST.md` in your repo root. Copy this checklist in:

```markdown
## Week 2 Day 4 Ship Checklist

- [ ] All pages share one external stylesheet (main.css)
- [ ] Colour palette uses CSS custom properties at :root
- [ ] Typography is consistent across all pages
- [ ] Portfolio pages render correctly at 320px, 768px, and 1280px
- [ ] Gallery page uses Grid with auto-fit and minmax
- [ ] Navigation bar uses Flexbox and works at all three breakpoints
- [ ] At least three elements have hover transitions
- [ ] No !important in the stylesheet
- [ ] Only one ID selector in the stylesheet
- [ ] All three AI Audit Comparisons entries are present
- [ ] AI_AUDIT.md is honest and detailed
- [ ] Repo is clean, pushed, and has conventional commit messages
```

Tick every honestly-completed box. Any unticked box requires a one-line note under it stating what and when you will fix it.

**Expected output:**
`CHECKLIST.md` with all boxes honestly checked.

### Task 5: Update the AI Audit

**What to do:**
Add one final Comparisons entry to `AI_AUDIT.md` for the hover transitions. Ask AI to write an alternative hover effect for one of your elements and compare it to yours.

**Expected output:**
`AI_AUDIT.md` now has at least 3 Comparisons entries total for Week 2.

## Stretch Goals (Optional - Extra Credit)

- Use `clamp(min, preferred, max)` for responsive typography on your headings so they scale smoothly without media queries.
- Add a `prefers-reduced-motion` media query that disables transitions for users who prefer reduced motion.
- Deploy your portfolio to GitHub Pages and include the live link in your README.

## Submission Requirements

- **What to submit:** Portfolio repo with all Week 2 deliverables, `CHECKLIST.md`, `AI_AUDIT.md`.
- **Where to submit:** Week 2 submissions channel
- **Deadline:** End of Day 4
- **Format:** All files in existing repo

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Breakpoints chosen from content, not guessed | 10 | `layout-notes.md` has three breakpoints with one sentence each defending the choice. |
| Mobile-first media queries | 20 | All media queries use `min-width`. Base styles look right at 320px. Layout adapts cleanly at tablet and desktop widths. |
| Responsive across all pages | 15 | Every page (index, bio, post, gallery, layout-lab) renders correctly at 320px, 768px, and 1280px. |
| Hover transitions | 10 | At least three interactive elements have smooth transitions. |
| Ship checklist honest and complete | 15 | Every box checked or annotated. |
| Three total Comparisons entries in audit | 15 | One from Day 1, one from Day 2, one from Day 4 at minimum. Each with both versions, trade-offs, and a defended pick. |
| Clean commit history | 10 | Three-plus commits for Day 4 with conventional messages. |
| Code quality | 5 | No `!important`. No unused CSS. No hardcoded colours outside variables. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Guessing breakpoints.** 768px and 1024px are historical ballpark numbers, not sacred. Always verify against your own content.
- **Mixing `min-width` and `max-width` queries.** Pick one direction and stick with it -- mobile-first (`min-width`) is the modern standard.
- **Over-animating.** Hover transitions should be under 250ms. Slower feels sluggish; faster feels jittery.

## Resources

- Week 2 AI boundaries: [../ai.md](../ai.md)
- MDN media queries: https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries
- MDN transitions: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions
- Josh Comeau: The Surprising Truth About Pixels and Accessibility
