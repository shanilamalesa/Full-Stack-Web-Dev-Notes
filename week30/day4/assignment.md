# Week 30 - Day 4 Assignment

## Title
Demo Rehearsal And The Capstone Story

## Overview
Day 4 is rehearsal day. You write the capstone story (the 10-minute talk), practice with a timer, fix anything awkward, and commit the final slide deck/outline.

## Learning Objectives Assessed
- Write a clear 10-minute engineering story
- Practice with a stopwatch
- Anticipate demo-day failures and have fallbacks
- Finalise the deck

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio this week:** 20% manual / 80% AI
**Habit:** Launch week. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Slide copy.
- **NOT ALLOWED FOR:** The narrative (that is you).
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Story outline

**What to do:**
`capstone/story.md`:

```markdown
1. The problem (1 min)
2. The tenant model (1 min)
3. The data model and RLS (2 min)
4. Integrations: USSD, WhatsApp, M-Pesa (3 min)
5. Live demo: signup -> book -> pay -> confirmation (2 min)
6. What I learned and what is next (1 min)
```

Fill in each section with talking points.

**Expected output:**
Story committed.

### Task 2: Fallback plan

**What to do:**
If the internet dies, Meta is down, or your M-Pesa sandbox breaks, you still need to demo. Record a high-quality screen capture of the end-to-end flow as `fallback.mp4`. Commit it (or a link).

**Expected output:**
Fallback ready.

### Task 3: Slide deck

**What to do:**
Either markdown slides (Marp, Slidev) or Google Slides. 10-12 slides max. One idea per slide. No code screenshots longer than 6 lines.

Save a PDF as `capstone/deck.pdf`.

**Expected output:**
Deck committed.

### Task 4: Rehearse with a timer

**What to do:**
Three rehearsals with a stopwatch. Target 9:00-10:30 minutes. Record the three times in `rehearsal-times.md` and note what you cut.

**Expected output:**
Times committed.

### Task 5: Freeze code

**What to do:**
Tag `v1.0.0-rc1` and push. Fix only blockers after this tag.

**Expected output:**
Tag pushed.

## Stretch Goals (Optional - Extra Credit)

- Rehearse to a friend/teammate and collect feedback.
- Record a dry-run video and watch yourself.
- Prep Q&A answers.

## Submission Requirements

- **What to submit:** Repo, story, deck, fallback, times, tag, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Story outline | 25 | Six sections. |
| Fallback recorded | 20 | Ready. |
| Slide deck | 25 | 10-12 slides. |
| Rehearsal with times | 20 | Three attempts. |
| rc1 tag | 5 | Pushed. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Demo without a fallback.** Wifi fails every time.
- **Dense slides.** One idea per slide.
- **Timing your first attempt in front of the audience.** Practice with a clock.

## Resources

- Day 4 reading: [Demo Rehearsal.md](./Demo%20Rehearsal.md)
- Week 30 AI boundaries: [../ai.md](../ai.md)
