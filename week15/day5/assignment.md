# Week 15 - Day 5 Assignment

## Title
Squash, Rebase Interactively, And A Clean PR History

## Overview
Week 15 Day 5 teaches interactive rebase. You will take a messy branch with five small commits and squash them into one clean commit before opening a PR. This is how senior engineers keep project history readable.

## Learning Objectives Assessed
- Use `git rebase -i` to squash commits
- Reword commit messages during an interactive rebase
- Understand when to squash and when to keep history
- Force-push safely with `--force-with-lease`

## Prerequisites
- Weeks 1-14 Day 5 completed
- Your Week 15 shop repo with real changes on a feature branch

## AI Usage Rules

**Ratio:** 35/65. **Habit:** Paper first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining the interactive rebase UI.
- **NOT ALLOWED FOR:** Running the commands for you.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Create a messy branch

**What to do:**
Create `feature/messy-cart` from main. Make at least 5 small commits (trivial changes are fine):

```bash
git checkout -b feature/messy-cart
echo "1" >> test.txt && git commit -am "wip"
echo "2" >> test.txt && git commit -am "more wip"
echo "3" >> test.txt && git commit -am "fix typo"
echo "4" >> test.txt && git commit -am "fix it"
echo "5" >> test.txt && git commit -am "actually working now"
git log --oneline
```

**Expected output:**
5 messy commits visible. Screenshot `day5-before.png`.

### Task 2: Interactive rebase to squash

**What to do:**
Run:

```bash
git rebase -i HEAD~5
```

An editor opens with the 5 commits listed as `pick`. Change all but the first to `squash` (or `s`):

```
pick abc123 wip
squash def456 more wip
squash ghi789 fix typo
squash jkl012 fix it
squash mno345 actually working now
```

Save and close. A second editor opens asking you to compose the combined message. Replace everything with one clean conventional-commits-formatted message:

```
feat: add cart state machine with localStorage

- useReducer-based cart reducer
- CartProvider with persistence across reloads
- Cart badge in header showing total count
```

Save and close.

**Expected output:**
`git log --oneline` now shows ONE commit. Screenshot `day5-after.png`.

### Task 3: Reword a single commit

**What to do:**
Create another small commit on top of your squashed one with a deliberately bad message:

```bash
echo "typo" >> test.txt && git commit -am "asdf"
```

Rebase and reword:

```bash
git rebase -i HEAD~1
```

Change `pick` to `reword` on that commit. Save. The commit-message editor opens. Write a real message. Save.

**Expected output:**
Commit message updated in place.

### Task 4: Force-push with --force-with-lease

**What to do:**
```bash
git push origin feature/messy-cart --force-with-lease
```

The `--force-with-lease` refuses to overwrite if someone else pushed to the branch since you last pulled. Safer than plain `--force`.

**Expected output:**
Push succeeds. Screenshot `day5-force-lease.png`.

### Task 5: When NOT to squash

**What to do:**
In `rebase-notes.md`, write 5-7 sentences answering:
- When is squashing appropriate? (Hint: "wip" commits on a feature branch.)
- When is squashing harmful? (Hint: public history on main.)
- Why is `--force-with-lease` safer than `--force`?
- What is the golden rule of rewriting history? (Never rewrite shared history.)

Your own words.

**Expected output:**
`rebase-notes.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Use `git rebase -i` to split a commit into two (the `edit` command).
- Use `git cherry-pick` together with interactive rebase to reorder commits.
- Research `git worktree` as a way to work on multiple branches without stashing.

## Submission Requirements

- **What to submit:** Repo, screenshots, `rebase-notes.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Messy branch with 5 commits | 10 | Verified via screenshot. |
| Successful squash to 1 commit | 30 | Clean log. Message is conventional. |
| Reword via interactive rebase | 15 | Commit message updated in place. |
| Force-push with --force-with-lease | 15 | Command used, not plain --force. |
| When-not-to-squash notes | 20 | Four questions answered honestly. |
| Clean commits | 10 | Conventional messages throughout. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using `--force` instead of `--force-with-lease`.** Force can overwrite teammates' work silently.
- **Squashing commits on main.** If other people pulled those commits, squashing makes their histories diverge.
- **Forgetting what the message should be after a squash.** Compose it thoughtfully -- this is the single commit that will appear in the project history forever.

## Resources

- Prior Day 5 assignments
- Week 15 AI boundaries: [../ai.md](../ai.md)
- Pro Git Chapter 7 Rewriting History: https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History
