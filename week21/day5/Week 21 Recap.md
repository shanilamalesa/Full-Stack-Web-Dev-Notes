# Week 21, Day 5: Week Recap

Cron is one of those topics that feels boring until you ship software that customers depend on. Then suddenly "what happens if this job does not run" matters a lot.

---

## What You Built

1. A centralised cron registry in `server/cron/registry.js`.
2. Five standard scheduled jobs: reconciliation, cleanup, reports, reminders, health.
3. `cron_runs` audit table.
4. Retry with exponential backoff for transient errors.
5. Postgres advisory locks for overlap prevention.
6. Silent-stoppage detection via a meta job.
7. Graceful SIGTERM shutdown.
8. Resumable chunked jobs with `FOR UPDATE SKIP LOCKED`.
9. Redis distributed locks with token-verified release.
10. Leader election for multi-process deployments.
11. A separate worker process script (`server/worker.js`).
12. An admin `/admin/cron` dashboard.

---

## Self-Review Questions

1. What does `0 */4 * * *` mean?
2. How does `pg_try_advisory_lock` differ from a regular table lock?
3. When should you retry a task and when should you not?
4. What is the silent stoppage failure mode and how do you detect it?
5. What does `FOR UPDATE SKIP LOCKED` do and why does it help parallelism?
6. What is `SET NX PX` in Redis, and what does each flag mean?
7. Why does a distributed lock release need to check the token?
8. When would you pick in-process cron over a separate worker process?
9. What are the two places Vercel Cron calls (in the example architecture)?
10. Why is the cron health meta-job not sufficient as a monitoring strategy?

Target: 8/10.

---

## Peer Coding Session

### Track A: Migrate one cron to Vercel Cron
Pick `daily-report`. Move it out of the in-process registry and into a Vercel Cron route. Verify it still fires and the report still arrives.

### Track B: Write a Healthchecks.io integration
Sign up for the free tier. Each cron pings the service on start and end. Configure it to page you if pings stop arriving. This is production-grade observability in 30 minutes.

### Track C: Chunked cleanup
Write a cron that deletes old orders in chunks of 1000, logging progress, so it can run for hours without timing out.

### Track D: A cron dashboard in the Next.js shop
Port the `/admin/cron` page to the Next.js admin area, reusing the Server Component pattern from Week 15.

---

## Weekend Prep

A light weekend -- stitch cron into your shop. Eight jobs, eight commits. Details in `week21/weekend/project.md`.

Next week is **Project 4: Chama Savings Platform** -- the biggest application of Telegram + cron + payments + WhatsApp into one real product.
