# Week 30 - Day 5 Assignment

## Title
Demo Day, v1.0.0 Release, And The Marathon Retrospective

## Overview
The final day. You present the capstone, tag `v1.0.0`, trigger the semantic-release pipeline one more time, and write the marathon retrospective: 30 weeks in 1,000 words.

## Learning Objectives Assessed
- Present the capstone on demo day
- Cut a v1.0.0 release
- Write a honest retrospective of 30 weeks
- Close out the class-notes repo with a final push

## Prerequisites
- Days 1-4 completed

## AI Usage Rules

**Ratio this week:** 20% manual / 80% AI
**Habit:** Launch week. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Release notes drafting.
- **NOT ALLOWED FOR:** The retrospective. That is 100% yours.
- **AUDIT REQUIRED:** Yes. Final audit covers the whole marathon.

## Tasks

### Task 1: Deliver the demo

**What to do:**
Present the capstone. Follow your story from Day 4. Stay under 10 minutes. Have the fallback ready.

Record the presentation if possible.

**Expected output:**
Delivered.

### Task 2: Tag v1.0.0

**What to do:**
```bash
git tag -s v1.0.0 -m "Marathon capstone v1.0.0"
git push --tags
```

The publish workflow deploys the final version.

**Expected output:**
v1.0.0 deployed to prod.

### Task 3: Release notes

**What to do:**
`RELEASE_NOTES.md` for v1.0.0. List the features, the known issues, and the roadmap. Link to docs.

**Expected output:**
Committed.

### Task 4: 30-week retrospective

**What to do:**
`MARATHON_RETROSPECTIVE.md`, 800-1,200 words. Answer:
- What did the first week feel like vs. the last week?
- What was the single most valuable week?
- What was the week you were closest to quitting? Why did you not?
- What one technology did you think you knew but actually learned?
- What are you building next?
- What advice would you give to someone starting a Week 1?

No AI on this file. Zero. Write it yourself. It is yours.

**Expected output:**
Committed.

### Task 5: Close out

**What to do:**
- Update the root `README.md` of class-notes with a final Week 30 section
- Link to the capstone repo
- Push everything
- Delete the `dev` branch if you have one
- Archive or star the repo

Take a screenshot of the final repo state: `final-repo.png`.

**Expected output:**
Committed. Pushed. Done.

## Stretch Goals (Optional - Extra Credit)

- Post the retrospective publicly.
- Send the link to three people you respect.
- Start planning the next capstone.

## Submission Requirements

- **What to submit:** Repo with release notes, retrospective, screenshots, v1.0.0 tag, `AI_AUDIT.md`.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Demo delivered | 20 | Followed story. |
| v1.0.0 tag deployed | 20 | Prod updated. |
| Release notes | 15 | Features + known issues. |
| Retrospective | 35 | 800-1,200 words, personal, honest. |
| Final push and close-out | 5 | Repo clean. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Letting AI write the retrospective.** It knows nothing about your 30 weeks.
- **Skipping the release notes.** Future you will want them.
- **Not celebrating.** You finished a full-stack web dev marathon. Stop and notice.

## Resources

- Day 5 reading: [Demo Day.md](./Demo%20Day.md)
- Week 30 AI boundaries: [../ai.md](../ai.md)
