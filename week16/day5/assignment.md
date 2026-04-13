# Week 16 - Day 5 Assignment

## Title
Submodules, Dependencies, And Monorepo Hygiene

## Overview
Week 16 Day 5 tackles a common real-world Git problem: keeping multiple projects in sync via submodules or workspaces. Your shop depends on your Week 11-12 CRM backend. How do they share code? Submodules are one answer (though not always the best). You will practise the workflow end-to-end.

## Learning Objectives Assessed
- Add a git submodule to a repo
- Initialise and update submodules after cloning
- Understand when a submodule is a good idea vs when to use a monorepo or a published package
- Navigate a repo with submodules cleanly

## Prerequisites
- Weeks 1-15 Day 5 completed

## AI Usage Rules

**Ratio:** 30/70. **Habit:** AI as integration engineer. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Submodule command explanations.
- **NOT ALLOWED FOR:** Blindly running `git submodule` commands you do not understand.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Add a submodule

**What to do:**
Assume your shop needs a reusable library (e.g., a `shared-types` folder). Create a separate repo for it on GitHub. Then:

```bash
cd your-shop-repo
git submodule add https://github.com/YOUR_USERNAME/shared-types.git lib/shared-types
git commit -m "feat: add shared-types submodule"
```

Check that `.gitmodules` was created and committed.

**Expected output:**
`.gitmodules` exists. `lib/shared-types/` is a nested repo.

### Task 2: Clone with submodules

**What to do:**
In a different folder, clone your shop repo fresh:

```bash
git clone --recurse-submodules https://github.com/YOUR_USERNAME/your-shop.git /tmp/shop-clone
ls /tmp/shop-clone/lib/shared-types
```

Or if you forgot `--recurse-submodules`:

```bash
git clone https://github.com/YOUR_USERNAME/your-shop.git /tmp/shop-clone2
cd /tmp/shop-clone2
git submodule init
git submodule update
```

**Expected output:**
Submodule content is present in both clones.

### Task 3: Update the submodule

**What to do:**
Make a change in `shared-types` (push it upstream). Then in your shop:

```bash
cd lib/shared-types
git pull origin main
cd ../..
git add lib/shared-types
git commit -m "chore: update shared-types submodule"
git push
```

**Expected output:**
The shop now points at a newer commit of the submodule.

### Task 4: Submodule vs monorepo vs package

**What to do:**
In `submodule-notes.md`, write 6-8 sentences comparing three approaches:
1. **Git submodule** -- separate repos linked by a pinned commit
2. **Monorepo** -- one repo, multiple packages (npm workspaces)
3. **Published npm package** -- separate repo, published to npm

For each:
- When is it the right choice?
- What is the pain point?
- What is the hidden cost?

Your own words.

**Expected output:**
`submodule-notes.md` committed.

### Task 5: Clean up

**What to do:**
Remove the submodule (because you probably do not need it for Project 3):

```bash
git submodule deinit -f lib/shared-types
git rm -f lib/shared-types
rm -rf .git/modules/lib/shared-types
git commit -m "chore: remove shared-types submodule"
```

Document in `submodule-notes.md` why you are removing it (workspaces are better for this project).

**Expected output:**
Submodule cleanly removed.

## Stretch Goals (Optional - Extra Credit)

- Set up Nx or Turborepo for a more sophisticated monorepo setup.
- Publish one of your utility libraries to npm.
- Use `git subtree` as an alternative to submodules and document the difference.

## Submission Requirements

- **What to submit:** Repo with submodule history, `submodule-notes.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Submodule added and committed | 20 | `.gitmodules` present. |
| Fresh clone with recursive | 15 | Works both ways (with and without --recurse). |
| Update submodule pointer | 20 | Newer commit pinned in shop. |
| Comparison notes | 30 | Three approaches compared with honest trade-offs. |
| Clean removal | 10 | Submodule fully removed. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Forgetting `git submodule update` after cloning.** Your teammates will have empty submodule folders until they run it.
- **Treating submodules like regular folders.** They are separate repos. Changes must be pushed to the submodule's upstream, not the parent.
- **Using submodules when a monorepo is simpler.** Unless the teams are genuinely separate, a monorepo is usually better.

## Resources

- Prior Day 5 assignments
- Week 16 AI boundaries: [../ai.md](../ai.md)
- Pro Git Submodules: https://git-scm.com/book/en/v2/Git-Tools-Submodules
