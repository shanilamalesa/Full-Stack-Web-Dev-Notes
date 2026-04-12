# Week 21, Day 4: Choosing A Scheduler

By the end of today, you can explain the three main places scheduled work can live, pick the right one for each situation, and have tried at least two of them. You will also know when to reach for BullMQ (next week).

**Prior concepts:** in-process cron (Week 21 Days 1-3).

**Estimated time:** 2 hours

---

## The Three Places Scheduled Work Lives

### 1. In-process (inside your Express/Next app)

**Pros:** same codebase, shared db connection, simple deploy, no extra infra.
**Cons:** cron competes with HTTP for resources, deploys can kill in-flight jobs, requires leader election across multiple instances, hard to schedule across timezones.

**Pick when:** small app, single server, jobs finish quickly, team is small.

This is what you built in Days 1-3.

### 2. Separate worker process (same codebase, different deployment)

You split your repo into `server/web.js` (Express HTTP) and `server/worker.js` (cron). Both share `config/`, `services/`, `db/`. You deploy them as separate processes.

**Pros:** HTTP traffic and background work do not fight; you can scale them independently; deploys to web do not kill jobs in worker.
**Cons:** two deployments to manage; double infra cost; slightly more complicated logging.

**Pick when:** app has significant background work, team is okay managing two deploys, single database shared between the two.

```javascript
// server/worker.js
require("dotenv").config();
const { startAll } = require("./cron/registry");

console.log("Worker process starting");
startAll();

// Keep the process alive
process.on("SIGTERM", () => {
  console.log("Worker shutting down");
  process.exit(0);
});
```

In your `package.json`:

```json
{
  "scripts": {
    "start:web": "node server/web.js",
    "start:worker": "node server/worker.js"
  }
}
```

On deploy (Railway, Render, Fly, etc) you define two services: one running `start:web`, one running `start:worker`. They share env vars. They each get their own logs.

### 3. External scheduler (OS cron, Vercel Cron, GitHub Actions, Cloudflare Workers Cron)

Instead of running cron inside a Node process, you give an external scheduler a URL or CLI command, and it triggers your code on a schedule.

**OS cron (crontab):**

```cron
0 2 * * * cd /app && node scripts/reconcile.js
0 8 * * * curl -X POST https://yourapi.co.ke/api/cron/daily-report -H "Authorization: Bearer $TOKEN"
```

**Vercel Cron (if deployed on Vercel):**

```javascript
// app/api/cron/daily-report/route.js
import { NextResponse } from "next/server";
import { sendDailyReport } from "@/lib/tasks";

export async function POST(req) {
  const auth = req.headers.get("Authorization");
  if (auth !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }
  await sendDailyReport();
  return NextResponse.json({ ok: true });
}
```

And in `vercel.json`:

```json
{
  "crons": [
    { "path": "/api/cron/daily-report", "schedule": "0 8 * * *" }
  ]
}
```

**GitHub Actions cron:**

```yaml
name: Daily Report
on:
  schedule:
    - cron: "0 8 * * *"
jobs:
  daily-report:
    runs-on: ubuntu-latest
    steps:
      - run: curl -X POST https://yourapi.co.ke/api/cron/daily-report -H "Authorization: Bearer ${{ secrets.CRON_SECRET }}"
```

**Pros:** no uptime requirement on your app's worker; free on most platforms; observable externally; leader election is irrelevant.
**Cons:** every job is a network round trip; secrets must be managed in the external system; hard to share state with in-process jobs; alert when the external system is down.

**Pick when:** deploying to a platform that has native cron (Vercel, Cloudflare); running the cron from a machine that is not your main server; wanting observability in a separate system.

---

## The Matrix

| Situation | Recommendation |
|---|---|
| Small Kenyan shop, single server, jobs take <1 min | In-process |
| Medium app, 5 cron jobs, no separate workers yet | In-process, with a plan to move to workers |
| Long jobs (bulk SMS, data migrations) | Separate worker process |
| Deployed on Vercel | Vercel Cron for short jobs, separate worker for long |
| Hundreds of thousands of jobs per hour | BullMQ on a dedicated worker (Week 23) |
| Cross-timezone scheduling with strict SLAs | External scheduler (Temporal, Google Cloud Scheduler) |

For Project 4 next week, you use in-process. For Weeks 23-24 when we introduce BullMQ you migrate.

---

## A Quick Vercel Cron Example

If your shop is deployed on Vercel, add a cron that pings your reconciliation endpoint:

```javascript
// app/api/cron/reconcile/route.js
import { NextResponse } from "next/server";

export async function GET(req) {
  const auth = req.headers.get("Authorization");
  if (auth !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  // Call your Express server which holds the actual logic
  const res = await fetch(`${process.env.CRM_SERVER_URL}/api/internal/reconcile`, {
    method: "POST",
    headers: { "X-Internal-Secret": process.env.INTERNAL_SECRET },
  });

  return NextResponse.json({ triggered: res.ok });
}
```

```json
// vercel.json
{
  "crons": [
    { "path": "/api/cron/reconcile", "schedule": "0 2 * * *" }
  ]
}
```

Vercel calls your API route at 2am, which calls your Express server, which runs the real logic. Two hops, but you get Vercel's reliability and observability for free.

This is a perfectly reasonable architecture for small apps. The "Next.js app as thin cron trigger, Express app as business logic" pattern appears in a lot of real production systems.

---

## Checkpoint

1. The in-process cron from Days 1-3 still works.
2. You have split the code so `node server/worker.js` runs only the cron tasks without binding port 5000.
3. You have tried Vercel Cron locally with `vercel dev` (or equivalently an OS crontab entry that curls your API).
4. You can explain, in one sentence each, when to pick in-process vs separate worker vs external scheduler.
5. You picked the approach for Project 4 next week (hint: in-process).

Commit:

```bash
git add .
git commit -m "feat: split worker process and example external cron"
```

---

## What You Learned

- In-process cron is simplest but shares resources with HTTP.
- Separate worker processes split concerns cleanly.
- External schedulers remove the uptime requirement from your worker.
- Vercel Cron, OS cron, and GitHub Actions cron each fit specific needs.
- The "cron trigger + business logic API" pattern is common in production.

Tomorrow is the recap and a lighter weekend that stitches cron into your shop.
