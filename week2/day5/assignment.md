# Week 2 - Day 5 Assignment

## Title
Pull Requests, Code Review, and the GitHub Collaboration Workflow

## Overview
Last week's Day 5 you learned branches, merges, and merge conflicts all locally. This week you take the workflow to the cloud: pull requests, code review, and the collaboration rhythm every software team uses on GitHub. You will open a pull request against your own repo, request a review, leave comments on yourself, merge via the PR, and practise the `pull-edit-push` cycle that future classmates will use with you.

## Learning Objectives Assessed
- Open a pull request on GitHub with a clear title and description
- Use the GitHub review UI to leave line-level comments
- Merge a pull request and delete the branch cleanly
- Fetch and pull changes that someone else pushed
- Write a meaningful PR description using a standard template

## Prerequisites
- Completed Week 1 Day 5 (branches and merges)
- Completed Week 2 Day 1-4 assignments
- A partner in the cohort willing to exchange one PR review with you (or a second GitHub account you own)

## AI Usage Rules

**Ratio this week:** 85% manual / 15% AI
**Habit:** Compare and contrast. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Asking AI to explain Git error messages you read carefully. Asking AI about PR description best practices after writing your own first draft.
- **NOT ALLOWED FOR:** Generating PR descriptions, commit messages, or review comments for you.
- **AUDIT REQUIRED:** Yes, a final entry in `AI_AUDIT.md` for any Git/GitHub question you asked AI this week.

## Tasks

### Task 1: Create a feature branch and commit work to it

**What to do:**
In your portfolio repo, create a feature branch for a new page:

```bash
git checkout main
git pull
git checkout -b feature/contact-page
```

Create a new file `contact.html` with:
- Valid HTML5 boilerplate
- A `<form>` with `name`, `email`, `subject` (select), `message` (textarea), and a submit button
- All inputs with associated `<label>` tags

Style it using your existing `main.css`.

Commit in at least two steps:

```bash
git add contact.html
git commit -m "feat: add contact page skeleton"

# After styling
git add styles/main.css
git commit -m "style: contact form layout and inputs"
```

**Expected output:**
Two commits on `feature/contact-page`. Branch pushed to GitHub:

```bash
git push -u origin feature/contact-page
```

### Task 2: Open a pull request on GitHub

**What to do:**
1. Visit your repo on github.com. GitHub will show a banner: "Your recently pushed branches: feature/contact-page -> Compare & pull request". Click it.
2. Fill in the PR title using the conventional prefix: `feat: add contact page`
3. Fill in the description using this template:

```markdown
## What

Adds a new `contact.html` page with a form containing name, email, subject, and message fields.

## Why

Week 2 Day 5 assignment -- learning PR workflow. Also builds toward the weekend portfolio completeness.

## How to test

1. Open contact.html in Live Server
2. Verify the form renders
3. Verify inputs have labels (use keyboard Tab key to check focus order)

## Checklist

- [x] HTML5 validator passes
- [x] Form has labels
- [x] Styled with existing main.css
- [ ] Review comments addressed
```

4. Click "Create pull request".

**Expected output:**
A pull request open on your GitHub repo with the template filled in. Screenshot named `day5-pr-opened.png` in `assets/`.

### Task 3: Request a review from a peer (or self-review)

**What to do:**
**Option A (preferred):** Ask a cohort mate to review your PR. Provide them with the PR URL. They should leave at least two line-level comments.

**Option B:** Self-review. Open the "Files changed" tab in the PR. Click the `+` button next to any line of your own code and leave a comment. Leave at least two comments on yourself -- one positive ("I like this approach because...") and one critical ("I should rename this class because...").

**Expected output:**
At least two review comments visible on the "Files changed" tab. Screenshot `day5-review.png`.

### Task 4: Address review comments in new commits

**What to do:**
For each review comment, either:
- Make the change and commit it (conventional prefix: `refactor:` or `fix:` depending on what you are doing), or
- Reply explaining why you disagree (if reviewer is a peer) or resolve the comment with a note (if self-review)

Push the new commits. The PR will update automatically. Mark each comment thread as "Resolved" in GitHub's UI when done.

**Expected output:**
PR now has additional commits and all comment threads are resolved.

### Task 5: Merge the pull request

**What to do:**
On the PR page, click "Merge pull request". Pick "Create a merge commit" (not squash or rebase -- you want to see the branch history for learning). Confirm the merge.

After merging:

```bash
git checkout main
git pull
git branch -d feature/contact-page
```

**Expected output:**
PR shows as "Merged". Main branch locally has the new file. Feature branch deleted locally (GitHub keeps the historical ref).

### Task 6: Pull-edit-push cycle

**What to do:**
Pretend someone else (or actually a cohort mate) pushed a change to `main` while you were working. Simulate:
1. On github.com, click the "Edit" pencil icon on `README.md`, add a new line "## Week 2 Done", commit directly to main.
2. In your terminal on the main branch:

```bash
git status  # expects clean
git pull    # pulls the new README change
git log --oneline -5
```

See the new commit from GitHub come down.

**Expected output:**
`git log` shows the README commit from GitHub.

## Stretch Goals (Optional - Extra Credit)

- Set up a `CODEOWNERS` file that designates yourself as the owner of `styles/` directory.
- Add a branch protection rule that requires pull request reviews before merging to main (Settings > Branches).
- Use `gh` CLI instead of the web UI: `gh pr create`, `gh pr review`, `gh pr merge`. Document the commands in `notes.md`.

## Submission Requirements

- **What to submit:** Repo link, screenshots `day5-pr-opened.png` and `day5-review.png` in `assets/`, link to the merged PR.
- **Where to submit:** Week 2 submissions channel
- **Deadline:** End of Day 5
- **Format:** Link to the PR must be in a submitted document

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Feature branch with multiple commits | 15 | Branch created, at least two meaningful commits, pushed to GitHub. |
| Pull request opened with template | 20 | PR exists with conventional title, What/Why/How-to-test sections, and a checklist. |
| Review comments on PR | 15 | At least two line-level review comments visible (from peer or self-review). |
| Review comments addressed | 15 | Each comment either fixed with a new commit or resolved with a reply. No dangling unresolved threads. |
| PR merged and branch deleted | 10 | PR status: Merged. Feature branch deleted locally. Main branch contains the new file. |
| Pull-edit-push cycle demonstrated | 10 | `git pull` visibly brought down a change from GitHub. |
| Conventional commit messages throughout | 10 | All commits on the branch use proper prefixes. |
| Clean workflow | 5 | No force pushes. No accidental commits to main. No merge mess. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Pushing to main directly.** The whole point of Day 5 is branching. If you find yourself committing to main, switch branches first: `git checkout -b feature/...`.
- **Giant PRs.** A good PR is small enough to review in 10 minutes. If you are tempted to include 15 files of unrelated changes, split it into multiple PRs.
- **Merging without review.** Even in self-review, read your own diff before hitting merge. You will catch things you missed while coding.

## Resources

- Week 1 Day 5 assignment (foundation for branches and merges)
- Week 2 AI boundaries: [../ai.md](../ai.md)
- GitHub Pull Requests documentation: https://docs.github.com/en/pull-requests
- Conventional Commits: https://www.conventionalcommits.org/
- `gh` CLI: https://cli.github.com/
