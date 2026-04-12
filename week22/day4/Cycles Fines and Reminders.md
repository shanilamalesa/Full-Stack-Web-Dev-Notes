# Week 22, Day 4: Cycles, Fines, and Reminders

By the end of today, the chama runs itself. Cycles open and close automatically. Members who are late get fines calculated and applied. Reminders go out before the deadline. End-of-cycle reports land in the group.

**Prior-week concepts you will use today:**
- Cron registry (Week 21)
- Advisory locks (Week 21 Day 2)
- The chama schema and service from this week.

**Estimated time:** 3 hours

---

## End-Of-Cycle Cron

The core cron job: every day at 00:30 KE time, check every chama, and if today is their cycle day, close the old cycle and open a new one.

```javascript
// server/cron/tasks/chamaCycles.js
const { query, pool } = require("../../config/db");
const chamaService = require("../../services/chama/chamaService");

async function processCycles() {
  const today = new Date();
  const dayOfMonth = today.getDate();

  const { rows: chamas } = await query(
    "SELECT * FROM chamas WHERE cycle_day = $1",
    [dayOfMonth]
  );

  for (const chama of chamas) {
    try {
      await closeAndOpenCycle(chama);
    } catch (err) {
      console.error(`Cycle processing failed for ${chama.chat_id}:`, err);
    }
  }
}

async function closeAndOpenCycle(chama) {
  const client = await pool.connect();
  try {
    await client.query("BEGIN");

    // 1. Find the current open cycle
    const { rows: cycleRows } = await client.query(
      "SELECT * FROM cycles WHERE chama_id = $1 AND status = 'open' FOR UPDATE",
      [chama.chat_id]
    );
    const currentCycle = cycleRows[0];
    if (!currentCycle) {
      // First-ever cycle
      await client.query(
        `INSERT INTO cycles (chama_id, period_start, period_end, expected_total_cents)
         VALUES ($1, $2, $3, $4)`,
        [
          chama.chat_id,
          new Date().toISOString().slice(0, 10),
          new Date(Date.now() + 30 * 86400000).toISOString().slice(0, 10),
          chama.monthly_amount_cents * (await getMemberCount(client, chama.chat_id)),
        ]
      );
      await client.query("COMMIT");
      return;
    }

    // 2. Close the current cycle
    await client.query(
      "UPDATE cycles SET status = 'closed' WHERE id = $1",
      [currentCycle.id]
    );

    // 3. Apply fines for everyone who is short
    const { rows: shortfalls } = await client.query(
      `SELECT m.user_id, m.name,
              COALESCE(SUM(c.amount_cents) FILTER (WHERE c.status = 'confirmed'), 0) AS contributed,
              $2 AS expected
       FROM chama_members m
       LEFT JOIN contributions c ON c.member_user_id = m.user_id AND c.cycle_id = $1
       WHERE m.chama_id = $3 AND m.left_at IS NULL
       GROUP BY m.user_id, m.name`,
      [currentCycle.id, chama.monthly_amount_cents, chama.chat_id]
    );

    for (const s of shortfalls) {
      const shortfall = parseInt(s.expected, 10) - parseInt(s.contributed, 10);
      if (shortfall <= 0) continue;
      const fine = Math.round(shortfall * (chama.fine_percent / 100));
      await client.query(
        `INSERT INTO fines (cycle_id, member_user_id, amount_cents, reason)
         VALUES ($1, $2, $3, $4)`,
        [currentCycle.id, s.user_id, fine, `Shortfall on cycle ${currentCycle.period_start}`]
      );
    }

    // 4. Open a new cycle
    const newPeriodStart = new Date().toISOString().slice(0, 10);
    const newPeriodEnd = new Date(Date.now() + 30 * 86400000).toISOString().slice(0, 10);
    await client.query(
      `INSERT INTO cycles (chama_id, period_start, period_end, expected_total_cents)
       VALUES ($1, $2, $3, $4)`,
      [chama.chat_id, newPeriodStart, newPeriodEnd, chama.monthly_amount_cents * (await getMemberCount(client, chama.chat_id))]
    );

    await client.query("COMMIT");

    // 5. Announce in the group (outside the transaction)
    await sendCycleSummary(chama, currentCycle, shortfalls);
  } catch (err) {
    await client.query("ROLLBACK");
    throw err;
  } finally {
    client.release();
  }
}

async function getMemberCount(client, chamaId) {
  const { rows } = await client.query(
    "SELECT COUNT(*)::int AS count FROM chama_members WHERE chama_id = $1 AND left_at IS NULL",
    [chamaId]
  );
  return rows[0].count;
}

module.exports = { processCycles };
```

Long function, but each step is deliberate:

1. Find the open cycle.
2. Close it.
3. Compute shortfalls for every member and write fines.
4. Open the new cycle with the expected total based on current membership.
5. Announce (after COMMIT -- the announcement is a side effect).

Wrap the whole thing in a transaction so a crash mid-way does not leave you with two open cycles.

### The summary announcement

```javascript
async function sendCycleSummary(chama, cycle, shortfalls) {
  const totalCollected = shortfalls.reduce((sum, s) => sum + parseInt(s.contributed, 10), 0);
  const late = shortfalls.filter((s) => parseInt(s.contributed, 10) < chama.monthly_amount_cents);

  const lines = [
    `Cycle ${cycle.period_start} closed.`,
    `Total collected: KSh ${(totalCollected / 100).toLocaleString()}`,
    `Members on time: ${shortfalls.length - late.length} / ${shortfalls.length}`,
    "",
    `Late members:`,
    ...late.map((s) => `- ${s.name}: KSh ${((parseInt(s.expected, 10) - parseInt(s.contributed, 10)) / 100).toLocaleString()} short`),
    "",
    `New cycle opened for ${new Date(cycle.period_end).toLocaleDateString("en-KE", { month: "long" })}.`,
  ];

  await telegramService.sendMessage(chama.chat_id, lines.join("\n"));
}
```

Members see the full group summary. Names of late contributors are public on purpose -- chamas are peer-accountability systems. If your chama prefers to keep this private, add a config flag.

---

## Reminder Cron (Day Before The Cycle)

```javascript
async function sendCycleReminders() {
  const tomorrow = new Date(Date.now() + 86400000).getDate();

  const { rows: chamas } = await query(
    "SELECT * FROM chamas WHERE cycle_day = $1",
    [tomorrow]
  );

  for (const chama of chamas) {
    // Find members who have not contributed for the current open cycle
    const { rows: late } = await query(
      `SELECT m.user_id, m.name
       FROM chama_members m
       LEFT JOIN contributions c ON c.member_user_id = m.user_id
         AND c.cycle_id = (SELECT id FROM cycles WHERE chama_id = $1 AND status = 'open')
         AND c.status = 'confirmed'
       WHERE m.chama_id = $1 AND m.left_at IS NULL
       GROUP BY m.user_id, m.name
       HAVING COALESCE(SUM(c.amount_cents), 0) < $2`,
      [chama.chat_id, chama.monthly_amount_cents]
    );

    if (late.length === 0) continue;

    // Post a gentle reminder in the group
    await telegramService.sendMessage(chama.chat_id,
      `Tomorrow is cycle day. These members still need to contribute:\n` +
      late.map((m) => `- ${m.name}`).join("\n") +
      `\n\nType /contribute now to avoid fines.`
    );

    // And privately DM each late member
    for (const m of late) {
      try {
        await telegramService.sendMessage(m.user_id,
          `Hi ${m.name}, reminder: tomorrow is the cycle day for ${chama.name}. Run /contribute to pay in.`
        );
      } catch (err) {
        // 403 means no private chat; group reminder already sent
      }
      await new Promise((r) => setTimeout(r, 200));
    }
  }
}
```

Two reminders -- public group nudge plus private DMs. Members can miss one but almost never both.

---

## Register The Crons

In `server/cron/registry.js`:

```javascript
{
  name: "chama-cycles",
  schedule: "30 0 * * *",          // 00:30 every day
  timezone: "Africa/Nairobi",
  run: require("./tasks/chamaCycles").processCycles,
},
{
  name: "chama-reminders",
  schedule: "0 18 * * *",          // 18:00 every day -- evening reminder
  timezone: "Africa/Nairobi",
  run: require("./tasks/chamaCycles").sendCycleReminders,
},
```

Two scheduled jobs, with advisory-lock protection so a slow run does not overlap the next fire.

---

## End-Of-Month Summary PDF

For the treasurer and for audit, generate a monthly PDF summary. Reuse `pdfkit` from Week 10:

```javascript
async function generateMonthlyPdf(chamaId, cycleId) {
  const PDFDocument = require("pdfkit");
  const stream = require("fs").createWriteStream(`reports/${chamaId}-${cycleId}.pdf`);
  const doc = new PDFDocument();
  doc.pipe(stream);

  // Title
  const { rows } = await query("SELECT name FROM chamas WHERE chat_id = $1", [chamaId]);
  doc.fontSize(20).text(`${rows[0].name} -- Monthly Report`);

  // Summary stats
  // Member-by-member breakdown
  // Fines
  // Payouts

  doc.end();
  return new Promise((resolve) => stream.on("finish", () => resolve(stream.path)));
}
```

At cycle close, generate the PDF and send it to the treasurer as a Telegram document:

```javascript
const path = await generateMonthlyPdf(chama.chat_id, currentCycle.id);
await bot.sendDocument(chama.treasurer_user_id, path, {}, {
  filename: `${chama.name}-${currentCycle.period_start}.pdf`,
});
```

The treasurer's monthly homework is now a one-button download.

---

## Checkpoint

1. Running `processCycles()` manually on a chama whose cycle_day is today closes the old cycle, writes fines, opens the new cycle, and announces the summary.
2. A member who did not contribute gets a fine row with the correct amount.
3. Running `sendCycleReminders()` on a chama whose next cycle is tomorrow sends the group nudge and private DMs.
4. Both cron jobs are registered and log to `cron_runs` on each fire.
5. A generated PDF arrives in the treasurer's private chat.

Commit:

```bash
git add .
git commit -m "feat: cycle processing fines and reminders via cron"
```

---

## What You Learned

- One transaction for cycle close + new cycle open = safe.
- Fines are computed, not stored as magic -- a shortfall * fine_percent formula.
- Two-channel reminders (group + private) improve compliance.
- PDF summaries close the loop for the treasurer.

Tomorrow: the treasurer's web dashboard and Friday peer review.
