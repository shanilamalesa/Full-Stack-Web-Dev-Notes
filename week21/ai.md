# Week 21 - AI Boundaries

**Ratio this week: 30% Manual / 70% AI**
**Habit introduced: "AI writes schedules, you decide on reliability."**
**Shift from last week: Back to 30/70 because cron is mostly config.**

Cron jobs are a configuration problem wearing a programming problem's clothes. Writing a cron expression is trivial; writing a job that survives failures, retries, overlapping runs, and distributed environments is the actual work. The habit: AI is great at the expression and the syntax; you are responsible for the reliability decisions.

---

## What You MUST Do Manually (30%)

### Day 1 -- Cron fundamentals
- Read the cron syntax yourself. `* * * * *` and each field's meaning. Practise with crontab.guru.
- Install `node-cron` manually. Run your first scheduled task (`every minute, log hello`). Watch it fire.
- Kill the process and restart it. Understand: the schedule does not persist across restarts unless you make it persist.

### Day 2 -- Failure handling and overlap
- Write a job that can fail. Catch the error; log it; do not crash the process.
- Write a lock mechanism so two copies of the same job cannot run at once (file lock, Redis lock, or DB row lock). Design on paper first.
- Design retries: exponential backoff, max attempts, dead-letter queue.

### Day 3 -- Long running jobs and distributed locks
- Move from node-cron to a distributed scheduler if needed (BullMQ preview).
- Understand the leader-election problem: if you have three instances, which one runs the cron?

### Day 4 -- Choosing a scheduler
- Compare: node-cron, BullMQ, Temporal, cloud-managed (AWS EventBridge). Make a decision for the shop and document why.

---

## What You CAN Use AI For (70%)

- **Cron expressions.** AI is excellent here.
- **Job scaffolding** after you have written one by hand.
- **Lock implementations** using Redis or DB primitives.
- **Dashboard for scheduled job status.**

Forbidden:
- Deciding the reliability trade-offs.
- Choosing the retry policy.
- Writing the lock logic without understanding it.

---

## Things AI Is Bad At This Week

- **Time zones.** AI often writes cron that runs in UTC when you wanted EAT (UTC+3). Specify explicitly.
- **Daylight saving in non-UTC zones.** Rare problem in Kenya, common problem elsewhere. Use UTC if you are unsure.
- **At-least-once vs exactly-once.** AI defaults to at-most-once. For financial cron (reconciliation), you want at-least-once with idempotency.

---

## Core Mental Models For This Week

- **Cron is at-most-once by default.** If a run fails, the next scheduled fire still happens; the failed run is lost unless you retry.
- **A lock prevents overlapping runs.** Mandatory for any job that takes longer than its schedule interval.
- **A distributed scheduler has one leader.** Leader election is the hard part.

---

## This Week's AI Audit Focus

List every cron job you scheduled, with:
- The expression
- The time zone
- Whether it has a lock
- Whether it has retries
- Who wrote the reliability logic

---

## Assessment

- Walk through one of your cron jobs end-to-end. Explain the retry policy.
- Live test: kill the job mid-run. Does the lock release? Does the next scheduled fire work?

---

## One Sentence To Remember

"The schedule is trivial. The reliability is the job."
