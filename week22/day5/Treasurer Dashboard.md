# Week 22, Day 5: The Treasurer Dashboard and Week Recap

By the end of today, the treasurer has a small web dashboard showing every chama they manage, current cycle state, member contributions, and a button to trigger end-of-cycle manually. This is a small Next.js feature that reuses the payment-shop's admin patterns.

**Prior-week concepts you will use today:**
- The Next.js admin area (Week 15, Day 4)
- Server Actions (Week 15, Day 3)
- The chama services from this week.

**Estimated time:** 2 hours dashboard + 2 hours recap/peer.

---

## Auth: Telegram Login

The treasurer is identified by their Telegram user id, not a username/password. For the dashboard, use Telegram's "Login Widget" -- a small button that opens a Telegram popup and returns a signed user profile your server can verify.

For today's scope, take a shortcut: let the treasurer generate a one-time link from the bot with `/dashboard`:

```javascript
// /dashboard command
async function dashboardCommand(bot, message) {
  const chama = await chamaService.findByChatId(message.chat.id);
  if (!chama || chama.treasurer_user_id !== message.from.id) {
    return bot.sendMessage(message.chat.id, "Only the treasurer can do that.");
  }

  const token = crypto.randomUUID();
  await query(
    `INSERT INTO dashboard_sessions (token, user_id, created_at, expires_at)
     VALUES ($1, $2, NOW(), NOW() + INTERVAL '1 hour')`,
    [token, message.from.id]
  );

  const url = `${process.env.PUBLIC_URL}/treasurer?token=${token}`;
  await bot.sendMessage(message.from.id, `Open your dashboard: ${url}\n(link expires in 1 hour)`);
}
```

Schema:

```sql
CREATE TABLE dashboard_sessions (
  token UUID PRIMARY KEY,
  user_id BIGINT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL,
  expires_at TIMESTAMPTZ NOT NULL,
  used_at TIMESTAMPTZ
);
CREATE INDEX idx_dashboard_sessions_expires ON dashboard_sessions(expires_at);
```

On the Next.js side, `/treasurer?token=...` verifies the token, sets an HttpOnly cookie (same pattern as Week 15 admin), and redirects. The cookie holds the `user_id`. Subsequent pages check the cookie.

This is the simplest reasonable auth: bot-authenticated, short-lived links, no passwords. It is also what WhatsApp Web uses (the QR code is a one-shot auth token).

---

## The Dashboard Page

```jsx
// app/treasurer/page.js
import { cookies } from "next/headers";
import { redirect } from "next/navigation";
import { query } from "@/lib/db";
import Link from "next/link";

export default async function TreasurerPage() {
  const token = cookies().get("treasurer_token")?.value;
  if (!token) redirect("/treasurer/login");

  const { rows: session } = await query(
    "SELECT user_id FROM dashboard_sessions WHERE token = $1 AND expires_at > NOW()",
    [token]
  );
  if (!session[0]) redirect("/treasurer/login");
  const userId = session[0].user_id;

  const { rows: chamas } = await query(
    "SELECT * FROM chamas WHERE treasurer_user_id = $1",
    [userId]
  );

  return (
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-2xl font-bold mb-6">Your Chamas</h1>
      <div className="space-y-4">
        {chamas.map((c) => (
          <Link key={c.chat_id} href={`/treasurer/${c.chat_id}`} className="block border rounded p-4 hover:shadow">
            <div className="font-medium">{c.name}</div>
            <div className="text-sm text-gray-600">
              KSh {(c.monthly_amount_cents / 100).toLocaleString()} per cycle on day {c.cycle_day}
            </div>
          </Link>
        ))}
      </div>
    </div>
  );
}
```

A tree of pages:

- `/treasurer` -- list of chamas the treasurer manages.
- `/treasurer/[chamaId]` -- one chama's current cycle, member list, contribution table.
- `/treasurer/[chamaId]/history` -- past cycles.

Each is a Server Component, queries Postgres directly, passes data to small Client Components for any interactive bits (the "Close Cycle Manually" button, for instance).

### A useful one-liner stats widget

```jsx
<div className="grid grid-cols-3 gap-4 mb-6">
  <Stat label="This cycle" value={`KSh ${(stats.collectedThisCycle / 100).toLocaleString()} / ${(stats.expectedThisCycle / 100).toLocaleString()}`} />
  <Stat label="Members paid" value={`${stats.contributors} / ${stats.totalMembers}`} />
  <Stat label="Outstanding fines" value={`KSh ${(stats.outstandingFines / 100).toLocaleString()}`} />
</div>
```

Three numbers, told simply. The treasurer's main question is "where are we this month?" -- this card answers it in one glance.

### Member contributions table

```jsx
<table className="w-full">
  <thead><tr><th>Member</th><th>This cycle</th><th>Total ever</th><th>Fines</th></tr></thead>
  <tbody>
    {members.map((m) => (
      <tr key={m.user_id} className={`border-t ${m.cycleContribution < chama.monthly_amount_cents ? "bg-red-50" : ""}`}>
        <td>{m.name}</td>
        <td>KSh {(m.cycleContribution / 100).toLocaleString()}</td>
        <td>KSh {(m.totalEver / 100).toLocaleString()}</td>
        <td>KSh {(m.outstandingFines / 100).toLocaleString()}</td>
      </tr>
    ))}
  </tbody>
</table>
```

Red row = late. One glance tells the treasurer who to nudge. The whole dashboard is "one glance" design -- treasurers are usually volunteers juggling a day job and a family. The faster they can act, the more they trust the tool.

### Manual trigger actions

```jsx
"use client";
import { closeCurrentCycle, addPayout } from "@/app/actions/chama";

<button onClick={async () => {
  if (!confirm("Close the current cycle now?")) return;
  await closeCurrentCycle(chama.chat_id);
  router.refresh();
}}>Close cycle manually</button>
```

The Server Action calls the same `closeAndOpenCycle` function the cron uses. Day 4 work, reused, behind a button.

---

## Week 22 Recap

### What you built
1. Full chama data model: chamas, members, cycles, contributions, fines, payouts.
2. `/setup`, `/join`, `/balance`, `/stats`, `/members`, `/contribute` commands.
3. Contributions via M-Pesa through the published payments package.
4. Outbox-driven announcements.
5. End-of-cycle cron with transactional fines + new cycle open.
6. Cycle reminders (group + private DMs).
7. Monthly PDF reports.
8. A small treasurer web dashboard with bot-token auth.

### Self-review
1. What are the six invariants of the chama data model?
2. Why is the announcement sent after COMMIT, not inside the transaction?
3. How does the outbox worker know whether an `order.paid` event belongs to a chama contribution or a shop order?
4. Why are late member names public in the group summary but private in the reminder flow?
5. Why does the treasurer's dashboard use bot-token auth instead of email/password?
6. Where does the atomic "only confirm if still pending" pattern from Week 18 show up in this week's code?

### Peer session tracks

- **A**: Add a `/payout` command for the treasurer to log money going out.
- **B**: Add multi-chama support -- one user can belong to several chamas and get separate summaries.
- **C**: Build an alternate Kiswahili language version of all commands using the `t()` helper.
- **D**: Add a "grace period" feature -- members can contribute up to 3 days late without a fine.

---

## Weekend Prep

The weekend is the Project 4 deliverable: ship the chama platform into a real group and run one full cycle. Brief in `week22/weekend/project.md`.

Next week is BullMQ and we start turning the payments+notifications layer into a real queue-based system.
