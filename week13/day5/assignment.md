# Week 13 - Day 5 Assignment

## Title
Changelogs, Release Notes, and Semantic Versioning In Practice

## Overview
Week 13 Day 5 rounds out your Git and release workflow. You will write a `CHANGELOG.md` following the Keep a Changelog format, pick a version number using SemVer, tag a release, and publish release notes on GitHub that customers or cohort mates can actually read.

## Learning Objectives Assessed
- Write a CHANGELOG.md that a non-engineer can read
- Pick the right SemVer bump (MAJOR/MINOR/PATCH) for changes
- Use git tag to mark releases
- Publish a GitHub release with formatted notes

## Prerequisites
- Weeks 1-12 Day 5 completed

## AI Usage Rules

**Ratio:** 45/55. **Habit:** Protocol first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Suggesting release notes wording after you listed the changes.
- **NOT ALLOWED FOR:** Deciding the version bump.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Write CHANGELOG.md

**What to do:**
Create `CHANGELOG.md` at the repo root following the Keep a Changelog format (https://keepachangelog.com):

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on Keep a Changelog,
and this project adheres to Semantic Versioning.

## [Unreleased]

## [0.3.0] - 2026-MM-DD

### Added
- USSD menu via Africa's Talking sandbox (Week 13)
- State machine with Redis-backed sessions
- Multi-language support (English, Kiswahili)
- `leads.source` column distinguishing channels

### Changed
- Leads dashboard shows source badges

### Fixed
- Duplicate phone numbers no longer create duplicate leads

## [0.2.0] - 2026-MM-DD (end of Week 12)

### Added
- Postgres migration from SQLite
- JWT auth with bcrypt password hashing
- Row-scoped leads by assigned user
```

List at least the changes from Week 11 and Week 12. Dates are the Friday of each week.

**Expected output:**
`CHANGELOG.md` committed.

### Task 2: Pick the right SemVer bump

**What to do:**
Your current version is whatever tag you last pushed. Decide the next version:
- MAJOR bump for breaking changes (e.g. changing an API response shape)
- MINOR for new features (adding USSD, adding Postgres)
- PATCH for bug fixes

Document your reasoning in `release-notes.md`. Since you added new features without breaking existing ones, a MINOR bump (0.2.0 -> 0.3.0) is probably right.

**Expected output:**
`release-notes.md` with the reasoning.

### Task 3: Tag and push the release

**What to do:**
```bash
git tag -a v0.3.0 -m "Week 13: USSD channel"
git push origin v0.3.0
```

**Expected output:**
Tag visible on GitHub.

### Task 4: Publish a GitHub release

**What to do:**
On GitHub, create a release from the tag. Paste the `### Added` / `### Changed` / `### Fixed` sections from your changelog. Add a small paragraph at the top summarising the highlight ("This release adds a working USSD channel ..."). Publish.

**Expected output:**
Screenshot `day5-release.png` of the published release page.

### Task 5: Reflection

**What to do:**
In `release-notes.md`, answer in 5-7 sentences:
- Who is the audience for a changelog? (Not just other devs.)
- What does a good release note feel like to read?
- What is the difference between a commit message and a release note?
- When would you skip a release entirely?

Your own words.

**Expected output:**
`release-notes.md` complete.

## Stretch Goals (Optional - Extra Credit)

- Automate changelog generation from conventional commits using `standard-version` or `release-please`.
- Publish a `CHANGELOG.md` for every service separately in your monorepo.
- Sign your tags with GPG.

## Submission Requirements

- **What to submit:** Repo with `CHANGELOG.md`, `release-notes.md`, GitHub release URL, screenshot.
- **Deadline:** End of Day 5

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| CHANGELOG.md well-formatted | 25 | Keep a Changelog structure. At least two versions documented. |
| Correct SemVer bump with reasoning | 15 | Justified in release-notes.md. |
| Annotated git tag pushed | 15 | Tag visible on GitHub. |
| GitHub release published | 20 | Screenshot confirms. |
| Reflection notes | 20 | Four questions answered honestly. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Writing the changelog to sound impressive instead of useful.** Users want to know "will this affect me?" -- not a list of your achievements.
- **Using MAJOR for every new feature.** MAJOR means "existing users must change their code". If no one has to change, MINOR is correct.
- **Copying commits verbatim.** Commits are for you. Release notes are for users. Translate between the two.

## Resources

- Prior Day 5 assignments
- Week 13 AI boundaries: [../ai.md](../ai.md)
- Keep a Changelog: https://keepachangelog.com/
- Semantic Versioning: https://semver.org/
