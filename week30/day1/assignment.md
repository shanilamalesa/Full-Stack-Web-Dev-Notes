# Week 30 - Day 1 Assignment

## Title
Testing And QA Sweep

## Overview
Week 30 is launch week. Today you do a full QA sweep: automated tests, manual test plan, and bug triage for anything that remains.

## Learning Objectives Assessed
- Run the full test suite and chase every red
- Execute a written manual QA checklist
- Triage bugs by severity
- Decide what blocks launch and what ships as a known issue

## Prerequisites
- Weeks 1-29 completed

## AI Usage Rules

**Ratio this week:** 20% manual / 80% AI
**Habit:** Launch week -- AI does everything except judgment. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Test case generation, bug description drafting.
- **NOT ALLOWED FOR:** Severity calls and launch blockers.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Green the suite

**What to do:**
Run `npm test` for api, workers, web, and isolation suite. Fix every failing test. Commit a separate test-fix commit per fix.

**Expected output:**
All green.

### Task 2: Manual QA plan

**What to do:**
`qa/manual-plan.md` with at least 25 test cases grouped by feature:
- Signup (valid, invalid subdomain, taken subdomain, weak password)
- Login
- Product CRUD
- Order flow including M-Pesa sandbox
- USSD end-to-end
- WhatsApp round trip
- Cross-tenant attempts

For each: steps, expected, actual, pass/fail.

**Expected output:**
Executed, committed.

### Task 3: Bug triage

**What to do:**
Every failure becomes a GitHub issue labelled `bug`, severity `blocker`/`high`/`medium`/`low`. Blockers must be fixed before Day 3. Mediums and lows can ship as known issues.

**Expected output:**
Issues opened.

### Task 4: Load spot-check

**What to do:**
Use `autocannon` or `wrk` on the main endpoints for 30 seconds. Record p50/p95/p99 in `load-check.md`. Not a real load test; just a sanity check.

**Expected output:**
Numbers committed.

### Task 5: Launch blockers decision

**What to do:**
`launch-blockers.md`: list the exact issues that must be closed to launch. Keep it short. Anything not on this list ships.

**Expected output:**
Committed.

## Stretch Goals (Optional - Extra Credit)

- Add Playwright end-to-end tests for the top three flows.
- Add chaos tests that kill Redis mid-run.
- Record the full manual QA as a screencast.

## Submission Requirements

- **What to submit:** Repo, QA plan, issues, load check, blockers, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Suite green | 25 | Zero failing tests. |
| Manual QA plan | 25 | 25+ cases executed. |
| Bug triage | 20 | Issues opened and labelled. |
| Load check | 15 | p50/p95/p99 recorded. |
| Launch blockers | 10 | Short list. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Ignoring flaky tests.** Fix or delete.
- **Treating every bug as a blocker.** You never ship.
- **Load testing as a one-shot panic.** Have baselines.

## Resources

- Day 1 reading: [Testing and QA.md](./Testing%20and%20QA.md)
- Week 30 AI boundaries: [../ai.md](../ai.md)
