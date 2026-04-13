# Week 30 - Day 2 Assignment

## Title
Documentation And Runbook

## Overview
Day 2 writes the documentation your future self (and first users) will need: a README, a runbook for operations, and an API reference. No shortcuts.

## Learning Objectives Assessed
- Write a project README that onboards a new dev in 10 minutes
- Write a runbook with real incident responses
- Generate an API reference from your Week 27 api.md
- Keep docs close to code

## Prerequisites
- Day 1 completed

## AI Usage Rules

**Ratio this week:** 20% manual / 80% AI
**Habit:** Launch week. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Prose drafting.
- **NOT ALLOWED FOR:** Runbook decisions (you write those).
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Onboarding README

**What to do:**
`README.md` at the root of the capstone:
- Project description, one paragraph
- Quickstart: clone, `./scripts/dev-up.sh`, open URL
- Architecture: one paragraph + diagram
- Links to ADRs, api.md, runbook

**Expected output:**
A new dev clone -> running in 10 minutes.

### Task 2: Runbook

**What to do:**
`docs/RUNBOOK.md` with sections:

```markdown
## Common incidents

### Payments callback not updating wallet
Check `payment_attempts` for `pending` older than 10 min. Run reconcile script.

### Worker queue depth > 100
Check worker logs. Restart worker. Alert if recurring.

### 500s on a single tenant
Check RLS context is set in the handler.

### Database running out of connections
Check idle connections. Restart the API.
```

At least five incidents with concrete steps.

**Expected output:**
Committed.

### Task 3: API reference

**What to do:**
`docs/api-reference.md`: expand the Week 27 api.md with:
- Curl example per endpoint
- Error codes
- Auth requirements

**Expected output:**
Committed.

### Task 4: Environment and secrets doc

**What to do:**
`docs/secrets.md` describes every env var, what it does, where it is set, and whether it is rotated.

Never commit real values.

**Expected output:**
Committed.

### Task 5: Readme polish

**What to do:**
Add a simple architecture diagram to the root README. ASCII or image. Keep it accurate.

**Expected output:**
README updated.

## Stretch Goals (Optional - Extra Credit)

- Generate OpenAPI from zod schemas.
- Publish a small docs site with MkDocs or Docusaurus.
- Add a CONTRIBUTING.md.

## Submission Requirements

- **What to submit:** Repo with README, runbook, api reference, secrets doc, `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| README | 25 | 10-minute onboarding. |
| Runbook | 30 | Five real incidents. |
| API reference | 20 | Every endpoint covered. |
| Secrets doc | 15 | Every env var listed. |
| Diagram | 5 | Accurate. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **README that does not actually work if followed.** Test it.
- **Runbook of platitudes ("check the logs").** Be specific.
- **Docs that drift.** Put them next to code.

## Resources

- Day 2 reading: [Documentation and Runbook.md](./Documentation%20and%20Runbook.md)
- Week 30 AI boundaries: [../ai.md](../ai.md)
