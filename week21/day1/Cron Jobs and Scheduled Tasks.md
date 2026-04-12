# Week 21, Day 1: Cron Jobs and Scheduled Tasks

By the end of today, you understand cron syntax, have a small scheduled task infrastructure running inside the Express server, and know the five common scheduled jobs that every real app ends up needing.

**Prior-week concepts you will use today:**
- `node-cron` preview from Week 20 Day 4
- The reconciliation worker pattern from Week 18 Day 4
- The outbox worker from Week 18 Day 2

**Estimated time:** 3 hours

---

## The Week Ahead

| Day | Focus |
|---|---|
| Day 1 (today) | Cron syntax, `node-cron`, five common scheduled jobs. |
| Day 2 | Failure handling, overlap prevention, monitoring. |
| Day 3 | Long-running jobs, distributed locks, leader election. |
| Day 4 | OS-level cron vs in-process cron -- when to pick which. |
| Day 5 | Recap. |
| Weekend | Pull everything into the shop: hourly reconciliation, daily reports, weekly cleanup. |

---

## What Cron Is

Cron is a convention for "run this thing on this schedule". Unix had `cron` in 1975. `node-cron` reimplements the same schedule syntax inside a Node process. The syntax:

```
* * * * *
| | | | |
| | | | +--- day of week (0-6, Sunday=0)
| | | +----- month (1-12)
| | +------- day of month (1-31)
| +--------- hour (0-23)
+----------- minute (0-59)
```

Examples:

- `0 * * * *` -- every hour on the hour.
- `0 8 * * *` -- every day at 08:00.
- `0 9 * * 1-5` -- every weekday at 09:00.
- `*/5 * * * *` -- every 5 minutes.
- `0 0 1 * *` -- first of every month at midnight.
- `0 2 * * 0` -- every Sunday at 02:00.

The `*` means "every". The numbers mean "exactly this". `*/5` means "every 5". `1-5` means "1 through 5". Comma-separated like `1,5,10` means "these specific values".

If you cannot remember a cron expression, https://crontab.guru is the canonical readable explainer.

---

## Five Jobs Every Real App Needs

After enough real apps, the same five jobs appear in almost every one:

1. **Reconciliation** -- compare external truth (provider records) to internal state. Nightly.
2. **Cleanup** -- delete old sessions, expired tokens, stale cache entries. Daily.
3. **Reports** -- summarise activity (daily/weekly orders, user counts, revenue). Daily at 8am for the owner's inbox.
4. **Reminders** -- notify users about upcoming deadlines (Week 20 Day 4). Variable.
5. **Health checks / external pings** -- warm caches, wake up sleeping services, verify third parties are up. Every few minutes.

All five fit into a tiny scheduler with consistent patterns.

---

## A Scheduled Task Registry

Rather than sprinkling `cron.schedule(...)` calls across the codebase, centralise them:

```javascript
// server/cron/registry.js
const cron = require("node-cron");
const tasks = require("./tasks");

const jobs = [
  {
    name: "reconciliation",
    schedule: "0 2 * * *",
    timezone: "Africa/Nairobi",
    run: tasks.runReconciliation,
  },
  {
    name: "cleanup-sessions",
    schedule: "0 3 * * *",
    timezone: "Africa/Nairobi",
    run: tasks.cleanupSessions,
  },
  {
    name: "daily-report",
    schedule: "0 8 * * *",
    timezone: "Africa/Nairobi",
    run: tasks.sendDailyReport,
  },
  {
    name: "overdue-reminders",
    schedule: "0 18 * * *",
    timezone: "Africa/Nairobi",
    run: tasks.sendOverdueReminders,
  },
  {
    name: "health-ping",
    schedule: "*/5 * * * *",
    timezone: "Africa/Nairobi",
    run: tasks.healthPing,
  },
];

function startAll() {
  for (const job of jobs) {
    cron.schedule(job.schedule, async () => {
      console.log(`[cron] ${job.name} starting`);
      const start = Date.now();
      try {
        await job.run();
        console.log(`[cron] ${job.name} finished in ${Date.now() - start}ms`);
      } catch (err) {
        console.error(`[cron] ${job.name} failed:`, err);
      }
    }, { timezone: job.timezone });
  }
}

module.exports = { startAll };
```

In `server/index.js`:

```javascript
require("./cron/registry").startAll();
```

Every scheduled job is now in one file. Adding one is adding an object to the list and writing its function in `tasks.js`.

---

## The Five Tasks

```javascript
// server/cron/tasks.js
const { query } = require("../config/db");
const recon = require("../services/reconciliation.service");
const whatsapp = require("../services/whatsapp.service");
const env = require("../config/env");

async function runReconciliation() {
  const yesterday = new Date(Date.now() - 86400000).toISOString().slice(0, 10);
  const result = await recon.reconcileDate(yesterday);
  if (result.mismatchCount > 0) {
    await whatsapp.sendTextMessage({
      to: env.SHOP_OWNER_PHONE,
      text: `Reconciliation for ${yesterday}: ${result.mismatchCount} mismatches. Check /admin/reconciliation.`,
    });
  }
}

async function cleanupSessions() {
  // Redis already TTLs them; Postgres sessions (if any) we clean here.
  const result = await query(
    `DELETE FROM user_sessions WHERE last_active_at < NOW() - INTERVAL '30 days'`
  );
  console.log(`[cron] Deleted ${result.rowCount} stale sessions`);
}

async function sendDailyReport() {
  const { rows } = await query(
    `SELECT
       COUNT(*)::int AS orders_today,
       COALESCE(SUM(subtotal_cents)::int, 0) AS revenue_cents,
       COUNT(*) FILTER (WHERE payment_method = 'mpesa')::int AS mpesa_count
     FROM orders
     WHERE created_at::date = CURRENT_DATE - INTERVAL '1 day'
       AND payment_status = 'success'`
  );
  const r = rows[0];

  const message = `Yesterday's report:
- Orders: ${r.orders_today}
- Revenue: KSh ${(r.revenue_cents / 100).toLocaleString()}
- M-Pesa: ${r.mpesa_count}`;

  await whatsapp.sendTextMessage({ to: env.SHOP_OWNER_PHONE, text: message });
}

async function sendOverdueReminders() {
  // From Week 20 Day 4
  const { rows } = await query(
    `SELECT phone, name FROM customers
     WHERE pending_payment = true AND reminded_at < NOW() - INTERVAL '2 days'`
  );
  for (const row of rows) {
    await whatsapp.sendTextMessage({
      to: row.phone,
      text: `Hi ${row.name}, friendly reminder about your pending payment.`,
    });
    await query("UPDATE customers SET reminded_at = NOW() WHERE phone = $1", [row.phone]);
    await new Promise((r) => setTimeout(r, 200));
  }
}

async function healthPing() {
  // Ping payment provider APIs to keep connections warm
  const providers = ["mpesa", "airtel", "stripe"];
  for (const p of providers) {
    try {
      // cheap call: get status of a known-old transaction
    } catch (err) {
      console.warn(`[cron] ${p} health check failed:`, err.message);
    }
  }
}

module.exports = {
  runReconciliation,
  cleanupSessions,
  sendDailyReport,
  sendOverdueReminders,
  healthPing,
};
```

Each task is a pure async function. The registry handles scheduling, logging, error catching. If a task fails, it logs and the next scheduled run tries again. No silent failures.

---

## Cron Logs Table

For auditability, persist every run:

```sql
CREATE TABLE cron_runs (
  id BIGSERIAL PRIMARY KEY,
  job_name TEXT NOT NULL,
  started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  finished_at TIMESTAMPTZ,
  status TEXT CHECK (status IN ('running', 'success', 'failed')),
  error_message TEXT,
  duration_ms INTEGER
);
CREATE INDEX idx_cron_runs_job_started ON cron_runs(job_name, started_at DESC);
```

Wrap the task runner:

```javascript
async function runJobWithLogging(job) {
  const { rows } = await query(
    `INSERT INTO cron_runs (job_name, status) VALUES ($1, 'running') RETURNING id`,
    [job.name]
  );
  const runId = rows[0].id;
  const start = Date.now();

  try {
    await job.run();
    const duration = Date.now() - start;
    await query(
      `UPDATE cron_runs SET finished_at = NOW(), status = 'success', duration_ms = $1 WHERE id = $2`,
      [duration, runId]
    );
  } catch (err) {
    const duration = Date.now() - start;
    await query(
      `UPDATE cron_runs SET finished_at = NOW(), status = 'failed', error_message = $1, duration_ms = $2 WHERE id = $3`,
      [err.message, duration, runId]
    );
    throw err;
  }
}
```

Now an admin page (`/admin/cron`) can query `cron_runs` and show:

- Last 50 runs per job.
- Success rate over the past week.
- Average duration.
- Any runs that took >10x the average (suspicious).

---

## Checkpoint

1. The five cron jobs are registered in one file.
2. Triggering `runReconciliation()` manually (from a Node REPL or a test route) logs to `cron_runs` and completes.
3. Changing `daily-report` to `* * * * *` (every minute) fires and sends a test message to the owner.
4. A failing task (throw a test error) logs to `cron_runs` with `status = 'failed'` and does not crash the server.
5. The next scheduled run of the failing task still fires -- cron does not stop after a failure.

Commit:

```bash
git add .
git commit -m "feat: cron registry with five scheduled tasks and run logging"
```

---

## What You Learned

- `node-cron` uses the standard Unix cron syntax plus a timezone option.
- Centralise scheduling in a registry; do not sprinkle `cron.schedule` everywhere.
- Five jobs appear in almost every real app.
- Log every run to a table for audit and monitoring.
- A failing task should not crash the server or skip future runs.

Tomorrow we deal with the harder problems: overlapping runs, long-running tasks, and how to tell when a job has silently stopped firing.
