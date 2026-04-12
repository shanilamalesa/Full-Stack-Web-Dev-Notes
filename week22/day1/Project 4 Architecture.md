# Week 22, Day 1: Project 4 -- Architecture

This week we build the Chama Savings Platform end-to-end. It combines everything from Phase 3: Telegram bots, payments, cron jobs, WhatsApp messages, and a small Next.js dashboard for the treasurer. By Friday night the bot is running in a real chama group. The weekend is polish and demo prep.

**Prior-week concepts you will use today:**
- Everything from Weeks 11-21 -- payments, cron, Telegram, WhatsApp, Postgres, Redis, Next.js.

**Estimated time:** 2 hours (design, not code).

---

## The Week Ahead

| Day | Focus |
|---|---|
| Day 1 (today) | Architecture and schema. No code. |
| Day 2 | Telegram bot core: contributions, balances, commands. |
| Day 3 | M-Pesa-powered contribution flow with the payments package. |
| Day 4 | Cron jobs: reminders, fines, monthly reports. |
| Day 5 | Treasurer dashboard + peer review. |
| Weekend | Ship in a real group. Project 4 deliverable. |

---

## What A Chama Is

A chama is a Kenyan savings group. A few friends, relatives, or colleagues pool money on a schedule (weekly or monthly). The pool either:

- **Rotates** -- the entire pot goes to one member per cycle (ROSCA, "merry-go-round").
- **Accumulates** -- the pool grows and funds joint investments (ASCA).
- **Hybrid** -- regular contributions, occasional payouts, some lending.

Each chama has a chairman, a secretary, and a treasurer. The treasurer tracks money in and out. In practice, the treasurer's job is a spreadsheet, phone calls to nag late payers, and sometimes cash they carry in their bag until the next meeting. The Chama Savings Platform automates all of that.

---

## User Stories

The minimum useful product:

1. **As a member**, I can contribute via M-Pesa through a Telegram bot without showing up to a physical meeting.
2. **As a member**, I can see my balance and the chama's total pot at any time.
3. **As the treasurer**, I can see who contributed this cycle, who has not, and send reminders.
4. **As the treasurer**, I can declare a cycle closed and track a payout.
5. **As a group**, we get automated fines for late contributions (2% of the due amount, say) without any human doing math.

There are a hundred more features you could add. Pick the first five and ship them.

---

## The Architecture

Four components already exist in your codebase from previous weeks:

```
  +----------------+     +-----------------+     +----------------+
  | Telegram bot   |     | Express server  |     |  Next.js admin |
  | (group chat)   |<--->|  - cron         |<--->|  (treasurer    |
  |                |     |  - payments     |     |   dashboard)   |
  +----------------+     |  - telegram     |     +----------------+
                         |  - outbox       |
                         +--------+--------+
                                  |
                         +--------+--------+
                         |   Postgres      |
                         |   + Redis       |
                         +-----------------+
```

All roads lead to Postgres. Telegram bot reads and writes. Cron reads and writes. Treasurer's dashboard reads and writes. The payments package is a library they all use.

---

## The Schema

```sql
-- A chama is a telegram group chat.
CREATE TABLE chamas (
  chat_id BIGINT PRIMARY KEY REFERENCES telegram_chats(id),
  name TEXT NOT NULL,
  monthly_amount_cents INTEGER NOT NULL CHECK (monthly_amount_cents > 0),
  fine_percent NUMERIC(5,2) DEFAULT 2.00,
  cycle_day INTEGER NOT NULL DEFAULT 1 CHECK (cycle_day BETWEEN 1 AND 28),
  treasurer_user_id BIGINT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE chama_members (
  chama_id BIGINT REFERENCES chamas(chat_id),
  user_id BIGINT NOT NULL,
  name TEXT NOT NULL,
  phone TEXT NOT NULL,
  joined_at TIMESTAMPTZ DEFAULT NOW(),
  left_at TIMESTAMPTZ,
  PRIMARY KEY (chama_id, user_id)
);

CREATE TABLE cycles (
  id BIGSERIAL PRIMARY KEY,
  chama_id BIGINT REFERENCES chamas(chat_id),
  period_start DATE NOT NULL,
  period_end DATE NOT NULL,
  status TEXT NOT NULL DEFAULT 'open'
    CHECK (status IN ('open', 'closed', 'cancelled')),
  expected_total_cents INTEGER NOT NULL,
  UNIQUE (chama_id, period_start)
);

CREATE TABLE contributions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  cycle_id BIGINT REFERENCES cycles(id),
  chama_id BIGINT,
  member_user_id BIGINT,
  amount_cents INTEGER NOT NULL CHECK (amount_cents > 0),
  mpesa_reference TEXT,
  status TEXT NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'confirmed', 'failed')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  confirmed_at TIMESTAMPTZ
);

CREATE TABLE fines (
  id BIGSERIAL PRIMARY KEY,
  cycle_id BIGINT REFERENCES cycles(id),
  member_user_id BIGINT NOT NULL,
  amount_cents INTEGER NOT NULL,
  reason TEXT NOT NULL,
  paid BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE payouts (
  id BIGSERIAL PRIMARY KEY,
  chama_id BIGINT,
  recipient_user_id BIGINT NOT NULL,
  amount_cents INTEGER NOT NULL,
  reason TEXT NOT NULL,
  paid_at TIMESTAMPTZ,
  created_by BIGINT NOT NULL
);
```

Six tables. Each represents something real in chama life. Let us walk each:

- **`chamas`** is the group. One row per chama. Chained to the Telegram chat so the bot knows which group is which chama.
- **`chama_members`** is the membership list.
- **`cycles`** is one contribution period (usually a month). An open cycle is the current one; closed cycles are history.
- **`contributions`** is a single payment a member made toward a cycle. Has the M-Pesa reference for audit.
- **`fines`** accumulates penalties for late payment.
- **`payouts`** is money going *out* of the chama to a member (for ROSCAs or group expenses).

---

## The Invariants

These are the rules the system must never break:

1. **One open cycle per chama at a time.** New cycles open when the previous one closes.
2. **A contribution's amount must exactly match the chama's monthly_amount** -- or be explicitly marked as a top-up or arrears. No random amounts.
3. **Sum of contributions minus sum of payouts = the chama's current balance.** Always. You derive it, you do not store it.
4. **A member can contribute to any open cycle their membership covers.** If they joined mid-cycle, the first cycle is prorated.
5. **Fines can be paid separately from contributions, or added to the next contribution.** Either way, the fine row ends up `paid = true`.
6. **Everything happens in a transaction.** No half-confirmed contributions.

Write these somewhere visible in your codebase (a comment at the top of the chama service, or in `ARCHITECTURE.md`). When you wonder "should I do X", check the invariants first.

---

## The Data Flow: Contribution Happy Path

```
1. Member types /contribute in the chama Telegram group (or private chat with the bot).
2. Bot responds with a keyboard: "Contribute KSh 500 (this month)" [preset] / "Custom amount".
3. Member clicks the preset.
4. Bot inserts a `contributions` row with status = 'pending' and calls the payments package.
5. Payments package initiates STK Push to member's phone.
6. Bot replies: "Sending STK prompt to your phone. Approve to complete."
7. Member enters PIN on their phone.
8. Daraja calls the webhook.
9. Payments package marks the payment successful and writes to the outbox.
10. Outbox worker picks up the event:
    - Updates contributions row to confirmed
    - Sends a Telegram message to the chama group: "Wanjiru just contributed KSh 500. Total for May cycle: KSh 4,500 / 5,000."
    - Sends a WhatsApp to the treasurer (if the chama wants it).
```

Ten steps. Each one you already built in previous weeks. Assembly is the work.

---

## The Data Flow: End of Cycle

Cron fires on the cycle day (say the 1st of each month):

```
1. Find all chamas with cycle_day = today.
2. For each chama:
   a. Find the open cycle.
   b. Sum confirmed contributions for the cycle.
   c. For members who contributed less than monthly_amount: create a fine.
   d. Close the cycle.
   e. Open a new cycle for the next month.
   f. Send a summary message to the Telegram group.
```

One cron job, one function, one transaction per chama. Day 4 builds it.

---

## A Bot-First Product Decision

Notice: there is no customer-facing frontend. Members interact with Telegram; the treasurer uses a small admin dashboard. That is the entire UI surface. Web pages for "your balance" exist but are a stretch goal.

Why bot-first: members already live in Telegram. Forcing them to open a new web app to see their balance is friction. Every "open the website" request is a friction point that drops 30% of users. A bot that answers `/balance` in the group chat they already check twice a day gets 100% use.

The treasurer is the only user who needs a dashboard, because the treasurer needs to see lots of things at once (all members, all contributions, all fines) and a conversation is the wrong shape for that view.

---

## Checkpoint

1. You can draw the architecture diagram on paper from memory.
2. You can explain each of the six tables and why they exist in their own words.
3. You can list the six invariants.
4. You can walk through the contribution happy path without looking.
5. You can walk through the end-of-cycle flow without looking.
6. You have not written any code yet.

Commit:

```bash
git add .
git commit -m "docs: add architecture sketch for project 4"
```

Add a file `server/chama/ARCHITECTURE.md` containing this content for reference.

---

## What You Learned

- Chamas are Kenyan savings groups with predictable structure.
- Six tables capture the whole data model.
- Invariants written down now save bugs later.
- The bot is the product; the web dashboard is a utility.
- Architecture before code -- always.

Tomorrow we start building the bot.
