# Week 21, Day 3: Long-Running Jobs and Distributed Locks

By the end of today, your scheduler can handle tasks that take hours, split them into resumable chunks, and coordinate across multiple Node processes so two servers do not both run the same job.

**Prior concepts:** Week 21 Days 1-2.

**Estimated time:** 2-3 hours

---

## Why Long-Running Matters

Most cron jobs finish in seconds. A few do not:

- **Reconciliation against a year's worth of orders** might take 30 minutes.
- **Sending a marketing blast to 50,000 customers** at 100ms per send is 80 minutes.
- **A data migration** copying millions of rows between tables can run for hours.
- **A video encoding pipeline** on a couple of uploaded videos.

These break the "run every 15 minutes" assumption. You need different patterns.

---

## Pattern 1: Chunked Resumable Jobs

Split the work into small chunks, store progress in the database, and resume from where you left off if the job is interrupted.

```javascript
// server/cron/tasks/bulkSms.js
const { query } = require("../../config/db");
const whatsapp = require("../../services/whatsapp.service");

async function runBulkSms(campaignId) {
  const { rows: campaignRows } = await query(
    "SELECT id, message FROM campaigns WHERE id = $1",
    [campaignId]
  );
  const campaign = campaignRows[0];
  if (!campaign) return;

  const CHUNK_SIZE = 50;

  while (true) {
    // Pick next chunk of pending deliveries
    const { rows: deliveries } = await query(
      `SELECT id, phone FROM campaign_deliveries
       WHERE campaign_id = $1 AND status = 'pending'
       ORDER BY id ASC
       LIMIT $2
       FOR UPDATE SKIP LOCKED`,
      [campaignId, CHUNK_SIZE]
    );

    if (deliveries.length === 0) break;

    for (const d of deliveries) {
      try {
        await whatsapp.sendTextMessage({ to: d.phone, text: campaign.message });
        await query("UPDATE campaign_deliveries SET status = 'sent', sent_at = NOW() WHERE id = $1", [d.id]);
      } catch (err) {
        await query("UPDATE campaign_deliveries SET status = 'failed', error = $1 WHERE id = $2", [err.message, d.id]);
      }
      await new Promise((r) => setTimeout(r, 100)); // 10/sec
    }
  }
}

module.exports = { runBulkSms };
```

Three patterns at work:

**Work ledger pattern.** `campaign_deliveries` is the work list. Each row tracks its own state (`pending`, `sent`, `failed`). The job reads the next chunk of `pending`, processes it, updates each row. Resume is automatic because we read the database on every chunk.

**`FOR UPDATE SKIP LOCKED`.** This clause lets us pick a chunk of rows, lock them so other workers cannot grab the same ones, and `SKIP LOCKED` means "if someone else already grabbed them, move past". It is how you parallelise a work queue across multiple processes without a race.

**Pacing.** 100ms between sends = 10/sec, safely under any rate limit.

This is a hand-rolled task queue. Week 23 we replace it with BullMQ (Redis-backed, with retries, priorities, dashboards). For now the database-backed version is enough to understand the shape.

---

## Pattern 2: Distributed Locks With Redis

Postgres advisory locks work great when there is one database. With two Node processes on two servers both using the same database, the lock still works. With different databases or different schemes, you need a different lock mechanism.

Redis has a well-known distributed lock pattern called **SET NX**:

```javascript
async function acquireLock(name, ttlMs = 60000) {
  const client = await getClient();
  const token = crypto.randomUUID();
  // SET key value NX PX milliseconds
  const result = await client.set(`lock:${name}`, token, { NX: true, PX: ttlMs });
  if (result === "OK") return token;
  return null;
}

async function releaseLock(name, token) {
  const client = await getClient();
  // Only release if our token matches -- prevents releasing someone else's lock
  const script = `
    if redis.call("get", KEYS[1]) == ARGV[1] then
      return redis.call("del", KEYS[1])
    else
      return 0
    end
  `;
  return client.eval(script, { keys: [`lock:${name}`], arguments: [token] });
}

async function runWithRedisLock(name, fn, ttlMs = 60000) {
  const token = await acquireLock(name, ttlMs);
  if (!token) {
    console.log(`[cron] ${name} skipped -- lock held`);
    return;
  }
  try {
    await fn();
  } finally {
    await releaseLock(name, token);
  }
}
```

Four things to notice.

**`SET NX`** -- "set if not exists". Atomic. Either your process sets the lock or it does not.

**`PX ttlMs`** -- the lock automatically expires after the TTL. If your process crashes without releasing, the next process can acquire it after the TTL.

**Token verification on release.** You lock, your TTL expires, another process acquires the lock, you finally finish -- if you released without checking the token, you would release someone else's lock. The Lua script makes "check and delete" atomic.

**TTL should be longer than your task takes.** If your job runs 2 hours and your TTL is 1 hour, another process picks up the lock and runs the same job in parallel. Size the TTL generously.

This is the "Redlock" pattern (minus the multi-instance Redis quorum, which is overkill for the scale this course targets). For one Redis and one job it is rock-solid.

---

## Pattern 3: Leader Election

When you have three Node processes and only one should run the scheduler (say, to avoid triple-sending the daily report), you need leader election. Redis locks give you this for free:

```javascript
// On boot:
async function becomeLeaderForever() {
  const token = crypto.randomUUID();
  const LEADER_KEY = "leader:cron";
  const LEADER_TTL = 30000; // 30 seconds

  async function renew() {
    const client = await getClient();
    const current = await client.get(LEADER_KEY);
    if (current === token) {
      await client.pExpire(LEADER_KEY, LEADER_TTL);
      return true;
    }
    const result = await client.set(LEADER_KEY, token, { NX: true, PX: LEADER_TTL });
    return result === "OK";
  }

  let isLeader = await renew();
  if (isLeader) {
    console.log("I am the leader; starting scheduler");
    require("./registry").startAll();
  }

  // Renew every 10 seconds
  setInterval(async () => {
    const stillLeader = await renew();
    if (stillLeader && !isLeader) {
      console.log("Became leader; starting scheduler");
      require("./registry").startAll();
    }
    isLeader = stillLeader;
  }, 10000);
}
```

Pattern:
- Every process tries to set `leader:cron` with its own token.
- Exactly one succeeds (atomic NX). That process is leader.
- The leader renews the key every 10 seconds (well under the 30-second TTL).
- If the leader dies, its renewals stop. The key expires. Other processes try to set it; one succeeds and becomes the new leader.

Failover is automatic within ~30 seconds.

For the Marathon we have one Express process, so leader election is mostly theoretical. Once Week 24 introduces multiple processes behind a load balancer, you implement this properly and avoid double-sending every scheduled job.

---

## When Not To Use In-Process Cron

In-process cron (`node-cron` inside your Express server) has one weakness: the cron is part of the web app. Heavy cron jobs can starve the HTTP handler. One server deploys during a long job and the job dies halfway.

The alternatives:

- **OS-level cron** on the host machine, calling your app via CLI commands. Clean separation.
- **A dedicated worker process** that runs only the cron and nothing else. Same codebase, separate deployment.
- **A managed scheduler** (GitHub Actions cron, Vercel Cron, Cloudflare Cron Triggers). No server to manage.

Tomorrow we weigh these options. For Project 4 next week, in-process cron is fine.

---

## Checkpoint

1. A bulk SMS campaign can be started, killed mid-way, and resumed without re-sending messages that were already sent.
2. Two processes both trying `acquireLock("daily-report")` result in exactly one acquiring.
3. Killing the process that holds the lock releases it after the TTL (verify with `redis-cli TTL lock:daily-report`).
4. A leader-election test: start three copies of the process, verify only one runs cron jobs, kill the leader, verify another takes over within 30 seconds.

Commit:

```bash
git add .
git commit -m "feat: resumable chunked jobs redis locks and leader election"
```

---

## What You Learned

- Long-running jobs live in a work-ledger table and process in chunks.
- `FOR UPDATE SKIP LOCKED` is how you parallelise a work queue safely.
- Redis `SET NX` gives you a distributed lock with TTL.
- Token-checked release prevents releasing someone else's lock.
- Leader election is a loop of lock renewals.
- In-process cron is fine for a single process; multi-process needs leader election or external scheduling.

Tomorrow we compare in-process cron to OS cron to managed schedulers and pick the right tool for each situation.
