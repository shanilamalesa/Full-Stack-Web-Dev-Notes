# Week 17 - Day 5 Assignment

## Title
Conventional Commits Linting And A Changelog Generator

## Overview
Week 17 Day 5 automates the changelog workflow. You install `standard-version` (or `release-please`) which reads your conventional commits and generates CHANGELOG entries automatically. You also tighten your commit lint rules so the messages that feed the changelog are always well-formed.

## Learning Objectives Assessed
- Configure commitlint with stricter rules
- Use `standard-version` to auto-generate CHANGELOG entries and bump versions
- Understand how conventional commits map to SemVer
- Tag releases automatically from commit history

## Prerequisites
- Week 12 Day 5 (commitlint basics) and Week 13 Day 5 (changelog basics)

## AI Usage Rules

**Ratio:** 30/70. **Habit:** Provider docs first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Config boilerplate.
- **NOT ALLOWED FOR:** Making up rules without reading the tool's docs.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Install standard-version

**What to do:**
```bash
npm install -D standard-version
```

Add to `package.json`:

```json
"scripts": {
  "release": "standard-version"
}
```

**Expected output:**
Script available.

### Task 2: First auto-generated release

**What to do:**
Ensure you have at least 3 commits since your last tag with conventional messages (`feat:`, `fix:`, `chore:`).

Run:

```bash
npm run release
```

It will:
1. Analyse commits since last tag
2. Bump version in `package.json` (MINOR for features, PATCH for fixes)
3. Update CHANGELOG.md
4. Create a git tag

Review the generated changelog. Commit and push:

```bash
git push --follow-tags
```

**Expected output:**
Screenshot `day5-release.png` of the new changelog entry and the tag.

### Task 3: Tighten commitlint rules

**What to do:**
Update `commitlint.config.js`:

```javascript
module.exports = {
  extends: ["@commitlint/config-conventional"],
  rules: {
    "type-enum": [2, "always", [
      "feat", "fix", "docs", "style", "refactor", "test", "chore", "ci", "perf"
    ]],
    "subject-case": [2, "always", "lower-case"],
    "subject-max-length": [2, "always", 72],
    "body-leading-blank": [2, "always"],
  },
};
```

Try committing messages that violate the rules. Verify they are rejected.

**Expected output:**
Stricter rules enforced.

### Task 4: BREAKING CHANGE demo

**What to do:**
Make a commit with a breaking change footer:

```
feat: change orders API response shape

BREAKING CHANGE: orders now return amount_cents instead of amount_shillings.
Migration: update your clients to read amount_cents.
```

Run `npm run release` again. The tool should bump a MAJOR version because of the `BREAKING CHANGE` footer.

**Expected output:**
Major bump in CHANGELOG. Screenshot `day5-major.png`.

### Task 5: Auto-release notes on GitHub

**What to do:**
Set up release-drafter or manually create a release on GitHub from your new tag. Paste the changelog section as the release notes.

**Expected output:**
GitHub release visible with automated notes.

## Stretch Goals (Optional - Extra Credit)

- Configure `release-please` as an alternative to `standard-version` for PR-based releases.
- Add a GitHub Action that runs `standard-version` on every merge to main.
- Sign your tags with GPG for verified releases.

## Submission Requirements

- **What to submit:** Repo with standard-version config, screenshots, CHANGELOG.md updated, `AI_AUDIT.md`.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| standard-version installed and working | 20 | Script runs and generates CHANGELOG. |
| Auto-generated release | 25 | Real version bump and tag. |
| Tightened commitlint rules | 20 | Bad messages rejected; good ones accepted. |
| BREAKING CHANGE demo | 20 | Major bump from footer. |
| GitHub release with notes | 10 | Published. |
| Clean commits | 5 | Conventional messages throughout. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Running standard-version with no commits since last tag.** It will error or no-op.
- **Editing CHANGELOG manually after the tool generates it.** The tool regenerates on next run. Make changes in commit messages instead.
- **Forgetting to push tags.** Use `git push --follow-tags`.

## Resources

- Prior Day 5 assignments
- Week 17 AI boundaries: [../ai.md](../ai.md)
- standard-version: https://github.com/conventional-changelog/standard-version
- Conventional Commits BREAKING CHANGE: https://www.conventionalcommits.org/en/v1.0.0/#examples
