# Week 4 - Day 5 Assignment

## Title
Git Stash, Cherry-Pick, and Recovering From a Broken Commit

## Overview
You have branches, merges, and PRs. Today you learn the "oh no" commands: `git stash` for saving uncommitted work in a hurry, `git cherry-pick` for grabbing a commit from another branch, and the recovery workflow when you commit to the wrong branch. Every engineer hits these situations in their first month at work. Hit them now, in a safe classroom, so they stop being scary.

## Learning Objectives Assessed
- Use `git stash` to save and restore uncommitted work
- Use `git cherry-pick` to copy a commit from one branch to another
- Recover when you accidentally commit to `main` instead of a feature branch
- Use `git reflog` to find "lost" commits

## Prerequisites
- Week 1-3 Day 5 assignments completed
- Your portfolio or `js-lab` repo with at least one working feature branch

## AI Usage Rules

**Ratio:** 75/25. **Habit:** Build first, enhance with AI. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining what a reflog entry means after you read it yourself.
- **NOT ALLOWED FOR:** Generating Git commands. Resolving "oh no" situations for you.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Stash and restore

**What to do:**
1. On `main`, make an edit to any file (for example, add a line to README.md). Do NOT commit.
2. Run:

```bash
git status          # shows the modified file
git stash           # stashes the change
git status          # clean working tree
git stash list      # shows the stash
```

3. Now make a different edit, commit it normally on main, then bring the stash back:

```bash
git stash pop       # reapplies the stash
git status          # shows the file modified again
```

4. Commit the stashed change:

```bash
git add README.md
git commit -m "docs: add stash demo line"
```

**Expected output:**
Screenshot of the sequence, saved as `day5-stash.png`.

### Task 2: Cherry-pick a commit

**What to do:**
1. Create two branches from main:

```bash
git checkout main
git checkout -b feature/alpha
# edit any file, commit
git commit -am "feat: add alpha feature"

git checkout main
git checkout -b feature/beta
# edit a different file, commit
git commit -am "feat: add beta feature"
```

2. While on `feature/beta`, cherry-pick the alpha commit:

```bash
git log feature/alpha --oneline     # copy the commit hash
git cherry-pick <hash>
```

3. Run `git log --oneline` on `feature/beta` -- you should see both commits.

**Expected output:**
Screenshot `day5-cherry.png` of the log on feature/beta showing both commits.

### Task 3: Recover from a wrong-branch commit

**What to do:**
Simulate the classic mistake: commit to main when you meant to commit to a feature branch.

```bash
git checkout main
# edit any file
git commit -am "feat: accidentally on main"
git log --oneline       # you see the commit on main
```

Now recover. Two steps:

1. Create a branch at the current commit so you do not lose the work:

```bash
git branch feature/recovery
```

2. Reset main back to before the commit:

```bash
git reset --hard HEAD~1
git log --oneline       # the commit is gone from main
```

3. Switch to the new branch and verify the commit is there:

```bash
git checkout feature/recovery
git log --oneline
```

**Expected output:**
Commit moved from main to feature/recovery. Screenshot `day5-recover.png`.

### Task 4: Use reflog to find a "lost" commit

**What to do:**
After Task 3, run:

```bash
git reflog
```

Read the output carefully. You will see every action you took, including the reset. Find the commit hash of the accidental commit. Explain in a comment in your `notes.md` how reflog shows "lost" commits that `git log` hides.

**Expected output:**
`notes.md` with a short explanation of what reflog does and why it saves you from mistakes.

### Task 5: Short write-up

**What to do:**
In `day5-recovery-notes.md`, write 4-6 sentences summarising:
- When to use `git stash` vs just committing
- What `cherry-pick` is useful for
- How to recover a commit made on the wrong branch
- What `git reflog` is good for

Your own words. No AI prose.

**Expected output:**
`day5-recovery-notes.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Practise `git stash pop` vs `git stash apply` and explain the difference in notes.
- Use `git reset --soft HEAD~1` instead of `--hard` and explain what changed.
- Set up a `git` alias: `git config --global alias.undo 'reset --soft HEAD~1'` and use it.

## Submission Requirements

- **What to submit:** Repo link, three screenshots, `day5-recovery-notes.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Stash workflow | 20 | Full stash-pop cycle demonstrated. Screenshot clear. |
| Cherry-pick | 20 | Alpha commit successfully copied to beta branch. Log shows both. |
| Wrong-branch recovery | 25 | Commit moved from main to recovery branch cleanly. Main is back to pre-commit state. |
| Reflog usage | 10 | Student read reflog output and explained it. |
| Recovery notes | 15 | Four questions answered in student's own words. |
| AI Audit | 5 | 15-minute rule entries as needed. |
| Clean commits | 5 | Conventional messages throughout. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using `git reset --hard` without understanding it.** `--hard` discards uncommitted changes. Always `git status` and `git stash` first if you have uncommitted work you want to keep.
- **Forgetting the branch you created before resetting.** If you skip the `git branch feature/recovery` step, the commit is "lost" to normal `git log` -- you need reflog to find it again.
- **Cherry-picking from a dirty working tree.** Commit or stash your current work before cherry-picking.

## Resources

- Week 1-3 Day 5 assignments (prior Git content)
- Week 4 AI boundaries: [../ai.md](../ai.md)
- Pro Git book, Ch. 7: https://git-scm.com/book/en/v2/Git-Tools-Stashing-and-Cleaning
- MDN-equivalent for Git: https://git-scm.com/docs/git-stash
