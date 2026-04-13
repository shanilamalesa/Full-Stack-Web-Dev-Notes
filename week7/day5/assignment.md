# Week 7 - Day 5 Assignment

## Title
Issues, Labels, Milestones -- Project Management On GitHub

## Overview
Week 7 Day 5 moves beyond Git itself into GitHub project management. You will turn your Task Manager into a real project plan: create issues for each feature, label them, group them into a milestone, and work through one issue to completion via a linked PR. This is how real software teams track work.

## Learning Objectives Assessed
- Create GitHub issues with clear titles and descriptions
- Label and categorise issues
- Group issues into a milestone
- Link a PR to an issue with "Fixes #123" so it auto-closes on merge
- Use the GitHub Projects (beta or classic) board

## Prerequisites
- Weeks 1-6 Day 5 assignments completed
- A React Task Manager repo from Day 4

## AI Usage Rules

**Ratio:** 60/40. **Habit:** Build first, enhance with AI. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining what a "good issue" looks like after you read a couple of real ones.
- **NOT ALLOWED FOR:** Generating your issues.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Create five issues

**What to do:**
On your Task Manager repo, create five issues. Each one should describe a feature or improvement:

1. Add a filter (All / Active / Completed)
2. Persist tasks to localStorage
3. Add an "Edit task" button
4. Add keyboard shortcuts (Enter to add, Esc to cancel)
5. Refactor TaskManager into smaller components (TaskList, TaskItem)

For each issue:
- Clear title
- Short description (2-4 sentences)
- Acceptance criteria as a checklist

```markdown
## Description
Users should be able to filter tasks by status.

## Acceptance criteria
- [ ] Three filter buttons: All, Active, Completed
- [ ] Clicking a button updates the visible list
- [ ] The active filter is visually highlighted
```

**Expected output:**
Five issues visible in the Issues tab.

### Task 2: Label the issues

**What to do:**
Create (or use existing) labels: `feature`, `bug`, `refactor`, `good-first-issue`, `priority:high`, `priority:low`.

Assign at least one label to each issue. Use `good-first-issue` on the one easiest for a new contributor.

**Expected output:**
Labels visible on all 5 issues.

### Task 3: Create a milestone

**What to do:**
Create a milestone called "v1.0 Task Manager". Set a due date two weeks from today. Assign all 5 issues to the milestone.

**Expected output:**
Milestone visible with 5 issues assigned.

### Task 4: Work one issue end-to-end

**What to do:**
Pick the "Add a filter" issue. Implement it:

1. Create a branch `feature/filter-tasks`
2. Build the feature
3. Push the branch
4. Open a PR. In the PR description, include `Closes #1` (replacing 1 with the actual issue number)
5. Merge the PR. GitHub should automatically close the linked issue.

**Expected output:**
Issue auto-closed after merge. Screenshot `day5-autoclose.png`.

### Task 5: GitHub Projects board (or issue views)

**What to do:**
On the repo, create a simple Project board (the new GitHub Projects). Add all 5 issues. Organise into three columns: "To do", "In progress", "Done". Move the completed issue to "Done".

Alternative: if new Projects is blocked on your account, use the issue filters (?q=is:issue) to filter by label + milestone and take a screenshot showing the grouping.

**Expected output:**
Screenshot `day5-board.png` of the project or filtered view.

## Stretch Goals (Optional - Extra Credit)

- Pin your milestone to the top of the Issues tab.
- Set up an issue template so future issues automatically get the right structure.
- Close all remaining 4 issues over the weekend and re-take the screenshot.

## Submission Requirements

- **What to submit:** Repo link, links to the 5 issues, link to the milestone, link to the merged PR, screenshots.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Five issues with structure | 20 | Each issue has title, description, acceptance criteria. |
| Labels assigned | 10 | At least one label on each issue. |
| Milestone created | 10 | "v1.0 Task Manager" with all 5 issues assigned. |
| Issue worked end-to-end | 30 | Feature branch, PR with "Closes #N", merged, issue auto-closed. |
| Project board or filtered view | 15 | Visual organisation of the issues. Screenshot. |
| Clean commits | 10 | Conventional messages. |
| AI Audit | 5 | Any AI usage logged. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Writing vague issues.** "Make it better" is not actionable. Write issues that a stranger could pick up and understand.
- **Forgetting the "Closes" keyword.** Without `Closes #N` or `Fixes #N` in the PR description, the issue does not auto-close on merge.
- **Creating 30 issues before building anything.** Start with 5. Finish one. Then grow.

## Resources

- Prior Day 5 assignments (Weeks 1-6)
- Week 7 AI boundaries: [../ai.md](../ai.md)
- GitHub Issues documentation: https://docs.github.com/en/issues
- GitHub Projects documentation: https://docs.github.com/en/issues/planning-and-tracking-with-projects
- Linking PRs to issues: https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue
