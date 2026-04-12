# Week 21, Day 2: Failure Handling and Overlap Prevention

By the end of today, your cron jobs retry on failure, never run two copies of themselves in parallel, alert you when they are silently broken, and recover cleanly when the server restarts mid-run.

**Prior concepts:** Week 21 Day 1 registry.

**Estimated time:** 3 hours

---

## The Three Failure Modes

Scheduled jobs fail in three distinct ways:

1. **The job itself errors.** Code bug, network timeout, provider down. You catch it, log it, move on. The next scheduled run retries.
2. **The job runs successfully but too slow.** It is still running when the next scheduled fire happens. Now two copies of the job are running -- fighting each other for database rows, sending double emails, burning CPU.
3. **The job silently stops firing.** The process crashed, the server restarted, a deploy replaced the code. You notice weeks later because reports stopped arriving.

Each failure mode has a specific fix.

---

## Fix 1: Retry With Backoff

The easy case. Wrap the task in a retry helper:

```javascript
async function withRetry(fn, { retries = 3, delayMs = 1000 } = {}) {
  let lastErr;
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (err) {
      lastErr = err;
      console.warn(`Attempt ${i + 1} failed:`, err.message);
      await new Promise((r) => setTimeout(r, delayMs * Math.pow(2, i)));
    }
  }
  throw lastErr;
}

// Usage:
await withRetry(async () => recon.reconcileDate(yesterday), { retries: 3, delayMs: 2000 });
```

Exponential backoff (`delayMs * 2^i`) spreads the retries so a transient provider blip has time to clear. First retry at 2s, second at 4s, third at 8s. Total is 14 seconds of patience, which fits comfortably inside a cron window that runs every few minutes.

Do not retry **every** error. Some errors mean "do not try again" (validation, auth). Retry only transient errors -- network, 5xx, timeouts. A smart retry:

```javascript
function isTransient(err) {
  if (err.code === "ECONNRESET" || err.code === "ETIMEDOUT") return true;
  if (err.response?.status >= 500) return true;
  return false;
}

async function withSmartRetry(fn, opts) {
  let lastErr;
  for (let i = 0; i < opts.retries; i++) {
    try {
      return await fn();
    } catch (err) {
      if (!isTransient(err)) throw err;
      lastErr = err;
      await new Promise((r) => setTimeout(r, opts.delayMs * Math.pow(2, i)));
    }
  }
  throw lastErr;
}
```

---

## Fix 2: Overlap Prevention With A Database Lock

The right way to prevent two runs of the same job is a **database lock**. Postgres has advisory locks that are fast and release automatically when the connection drops:

```javascript
async function runWithLock(jobName, fn) {
  const lockKey = hashCode(jobName); // integer hash

  const client = await pool.connect();
  try {
    const { rows } = await client.query("SELECT pg_try_advisory_lock($1) AS acquired", [lockKey]);
    if (!rows[0].acquired) {
      console.log(`[cron] ${jobName} skipped -- already running`);
      return;
    }

    try {
      await fn();
    } finally {
      await client.query("SELECT pg_advisory_unlock($1)", [lockKey]);
    }
  } finally {
    client.release();
  }
}

function hashCode(str) {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    hash = ((hash << 5) - hash) + str.charCodeAt(i);
    hash |= 0;
  }
  return hash;
}
```

Three things worth understanding.

**`pg_try_advisory_lock` is non-blocking.** It returns true if the lock was acquired, false if another session holds it. The cron fire that finds the lock held simply skips.

**The lock is released automatically if the connection dies.** Crash mid-run? The TCP connection to Postgres closes, Postgres notices, the lock vanishes. The next cron fire can acquire it and try again. Zero manual cleanup.

**The lock key is an integer, so we hash the job name.** Same hash -> same lock. Different jobs get different hashes and do not block each other.

Wrap every task:

```javascript
cron.schedule(job.schedule, async () => {
  await runWithLock(job.name, () => runJobWithLogging(job));
}, { timezone: job.timezone });
```

---

## Fix 3: Silent Stoppage Detection

This is the hardest failure mode because there is no error to catch. The job just stopped firing. The only way to know is to check "was it supposed to have run by now?"

Add a meta-job that checks the other jobs:

```javascript
async function checkJobHealth() {
  const expectedIntervals = {
    "reconciliation": 24 * 3600 * 1000, // should run once per day
    "daily-report": 24 * 3600 * 1000,
    "cleanup-sessions": 24 * 3600 * 1000,
    "health-ping": 5 * 60 * 1000, // every 5 min
  };

  for (const [jobName, expectedInterval] of Object.entries(expectedIntervals)) {
    const { rows } = await query(
      `SELECT MAX(started_at) AS last_run FROM cron_runs WHERE job_name = $1 AND status = 'success'`,
      [jobName]
    );
    const lastRun = rows[0].last_run;
    const timeSince = lastRun ? Date.now() - new Date(lastRun).getTime() : Infinity;
    const gracePeriod = expectedInterval * 1.5;

    if (timeSince > gracePeriod) {
      await whatsapp.sendTextMessage({
        to: env.SHOP_OWNER_PHONE,
        text: `CRON ALERT: ${jobName} has not run successfully in ${Math.round(timeSince / 60000)} minutes`,
      });
    }
  }
}
```

Schedule this every 15 minutes:

```javascript
{ name: "check-job-health", schedule: "*/15 * * * *", run: tasks.checkJobHealth }
```

Now if any core job has not run in 1.5x its expected interval, you get a WhatsApp. The check itself is a cron job, so if *it* silently stops firing you are also hosed -- but that is a meta-problem you can solve by having the web dashboard read `cron_runs` on every load and alert on the page.

For real production, use an external monitoring service like Healthchecks.io -- you ping it from each cron, and it alerts you if pings stop arriving. 100% uptime on the monitor, because it is not your server.

---

## Graceful Shutdown

When you deploy a new version of the Express server, the old process gets SIGTERM. By default it just dies, and any cron job running inside drops half-way. That corrupts state.

Handle it:

```javascript
// server/index.js
const cronTasks = new Set();

function registerRunningTask(taskPromise) {
  cronTasks.add(taskPromise);
  taskPromise.finally(() => cronTasks.delete(taskPromise));
}

process.on("SIGTERM", async () => {
  console.log("SIGTERM received, finishing cron tasks...");
  await Promise.allSettled([...cronTasks]);
  console.log("All cron tasks finished. Exiting.");
  process.exit(0);
});
```

Wrap the cron task runner:

```javascript
cron.schedule(schedule, () => {
  const taskPromise = runWithLock(jobName, runner);
  registerRunningTask(taskPromise);
}, opts);
```

Now a deploy waits (up to a limit) for in-flight tasks to finish before killing the process. Combined with the advisory lock, the new process picks up cleanly after the old one finishes.

---

## Admin Page: Cron Health

One page, one query:

```jsx
// app/admin/cron/page.js
import { query } from "@/lib/db";
import { requireAdmin } from "@/app/actions/auth";

export default async function CronPage() {
  await requireAdmin();
  const { rows: runs } = await query(
    `SELECT job_name, status, started_at, duration_ms, error_message
     FROM cron_runs
     ORDER BY started_at DESC
     LIMIT 50`
  );
  const { rows: summaries } = await query(
    `SELECT job_name,
            COUNT(*)::int AS runs,
            COUNT(*) FILTER (WHERE status = 'success')::int AS successes,
            MAX(started_at) AS last_run
     FROM cron_runs
     WHERE started_at > NOW() - INTERVAL '24 hours'
     GROUP BY job_name`
  );

  return (
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-2xl font-bold mb-6">Cron Jobs</h1>

      <section className="mb-8">
        <h2 className="font-semibold mb-3">Last 24h summary</h2>
        <table className="w-full text-sm">
          <thead><tr><th>Job</th><th>Runs</th><th>Success</th><th>Last run</th></tr></thead>
          <tbody>
            {summaries.map((s) => (
              <tr key={s.job_name} className="border-t">
                <td>{s.job_name}</td>
                <td>{s.runs}</td>
                <td>{s.successes}/{s.runs}</td>
                <td>{new Date(s.last_run).toLocaleString("en-KE")}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </section>

      <section>
        <h2 className="font-semibold mb-3">Recent runs</h2>
        <ul className="text-xs">
          {runs.map((r) => (
            <li key={r.id} className={`border-t py-1 ${r.status === "failed" ? "text-red-600" : ""}`}>
              {new Date(r.started_at).toLocaleString("en-KE")} - {r.job_name} - {r.status} - {r.duration_ms}ms
              {r.error_message && ` - ${r.error_message}`}
            </li>
          ))}
        </ul>
      </section>
    </div>
  );
}
```

Admins see at a glance whether everything is running healthily.

---

## Checkpoint

1. A task that throws a transient error retries up to 3 times with exponential backoff.
2. A task that throws a non-transient error does not retry.
3. Starting a long-running task and triggering a second cron fire within the duration logs "skipped -- already running".
4. Killing the process mid-task (via `kill -9`) releases the advisory lock (verify: the next run can acquire it).
5. Deliberately deleting a job from the registry causes the meta-check to fire a WhatsApp after 1.5x the expected interval.
6. `/admin/cron` shows the last 50 runs and the 24-hour summary.
7. Sending SIGTERM to the process waits for in-flight tasks to finish before exiting.

Commit:

```bash
git add .
git commit -m "feat: cron retries advisory locks and silent failure detection"
```

---

## What You Learned

- Retry only transient errors with exponential backoff.
- Postgres advisory locks are fast, automatic, and perfect for cron overlap prevention.
- Silent stoppage is detected by a meta-job checking last-run times.
- Graceful shutdown waits for in-flight tasks.
- An admin cron page is 50 lines of SQL away.

Tomorrow we deal with long-running jobs -- tasks that legitimately take hours -- and what distributed locking looks like when you have more than one Node process.
