# Week 21 Weekend: Stitch Cron Into The Shop

A lighter weekend. Add the five standard scheduled jobs to your Week 16 shop, plus three more specific ones. Each ships as its own commit.

**Estimated time:** 3-4 hours.

**Deadline:** Monday morning before Week 22.

---

## The Eight Jobs

### Core (from Week 21 Day 1)

1. **Reconciliation** -- runs nightly at 2am, calls the Week 18 reconcile function, WhatsApps the owner if mismatches > 0.
2. **Cleanup** -- runs daily at 3am, deletes stale sessions and outbox rows older than 30 days.
3. **Daily report** -- runs at 8am, WhatsApps yesterday's order count and revenue.
4. **Overdue reminders** -- runs at 6pm, WhatsApps customers with pending COD orders older than 2 days.
5. **Health ping** -- runs every 5 min, checks all three payment providers.

### Shop-specific

6. **Low stock alert** -- runs hourly, WhatsApps the owner when any product hits stock < 5.
7. **Abandoned cart reminder** -- runs every 2 hours, WhatsApps customers whose cart has items older than 4 hours (requires tracking carts server-side -- optional if you want to skip).
8. **Weekly revenue report** -- runs Mondays at 9am, aggregates the previous week's orders by category and sends a PDF summary to the owner.

---

## Requirements

- All 8 jobs registered in `server/cron/registry.js`.
- Each job logs to `cron_runs`.
- Each job uses the advisory lock wrapper to prevent overlap.
- Each job handles transient errors with a 3-retry exponential backoff.
- The `/admin/cron` page shows the last 24 hours of runs for each.
- A `README.md` section documents the eight jobs with their schedules and what they do.
- At least one job (your choice) has a test that triggers it manually.

---

## Grading Rubric (50 pts)

| Area | Points |
|---|---|
| All 8 jobs wired and firing | 20 |
| Advisory lock and retry in every job | 10 |
| Admin cron dashboard | 5 |
| At least one test for a job | 5 |
| README quality | 5 |
| No regressions (shop still works) | 5 |

---

## Hints

**Trigger jobs manually from a test route.** Add `POST /api/cron/trigger/:jobName` (admin only) that manually fires a job. This is the fastest way to verify the job works without waiting for the schedule.

**Use short schedules during dev.** `*/1 * * * *` fires every minute. Great for debugging. Revert to the real schedule before committing.

**Pre-aggregate the weekly report.** Do not compute it inside the cron job. Have a separate `weekly_summaries` table populated by the reconciliation job, and the weekly report just reads it. Cron jobs should be fast.

**WhatsApp template limits.** If you are sending more than 250 template messages a day (reports, reminders, alerts), you may hit Tier 1 rate limits. Consolidate multiple alerts into one daily summary message instead.

Good luck. Week 22 is Project 4.
