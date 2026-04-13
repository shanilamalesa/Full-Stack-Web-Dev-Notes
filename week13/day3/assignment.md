# Week 13 - Day 3 Assignment

## Title
Connect USSD To The CRM Database

## Overview
Yesterday's state machine saves leads to memory only. Today you wire the "Save" step to your Postgres `leads` table from Week 12, reuse the same service layer, and verify that a USSD-originated lead appears alongside WhatsApp-originated leads in the dashboard.

## Learning Objectives Assessed
- Reuse existing service layer from one channel (WhatsApp) in another (USSD)
- Insert rows into the shared `leads` table with a different source
- Verify data shows up in the CRM dashboard
- Handle duplicate phone numbers gracefully

## Prerequisites
- Days 1-2 completed
- Week 12 backend with Postgres and `leads` repo in place

## AI Usage Rules

**Ratio:** 45/55. **Habit:** Protocol first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Dashboard tweaks to show the lead source column.
- **NOT ALLOWED FOR:** The phone-to-lead mapping bridge.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Add `source` column to leads

**What to do:**
```sql
ALTER TABLE leads ADD COLUMN source TEXT NOT NULL DEFAULT 'manual'
  CHECK (source IN ('whatsapp', 'ussd', 'manual'));
```

Apply the migration. Existing leads get `manual` by default.

**Expected output:**
`leads.source` column exists with correct constraint.

### Task 2: Wire the USSD save step

**What to do:**
In the `CONFIRM -> 1 (save)` path of your state machine, call your existing `leadsService.create` (or repository directly if you exposed one) with the collected data plus `source: 'ussd'`:

```javascript
if (input === "1") {
  try {
    await leadsRepo.upsertByPhone({
      wa_phone: session.data.phone,
      name: session.data.name,
      source: "ussd",
    });
    await redis.del(KEY(sessionId));
    return "END Lead saved. Asante.";
  } catch (err) {
    console.error("USSD save failed:", err);
    return "END Error saving. Please try again later.";
  }
}
```

Use `upsertByPhone` (you may need to add it) so repeated USSD sessions update an existing lead instead of creating duplicates.

**Expected output:**
A full USSD session creates a row in `leads` with `source = 'ussd'`.

### Task 3: Show source in the dashboard

**What to do:**
Update your React dashboard from Week 11 to show the `source` column with a small badge (blue for whatsapp, green for ussd, grey for manual). AI-assisted is fine.

**Expected output:**
Dashboard clearly shows which leads came from which channel.

### Task 4: Duplicate handling

**What to do:**
Run two USSD sessions for the same phone number. The second should update the lead (e.g., change the name) rather than create a duplicate.

Verify by querying Postgres:

```sql
SELECT wa_phone, COUNT(*) FROM leads GROUP BY wa_phone HAVING COUNT(*) > 1;
```

Should return zero rows.

**Expected output:**
No duplicates in `leads`.

### Task 5: End-to-end test

**What to do:**
Write a small integration test that:
1. POSTs three USSD callbacks (welcome, name, phone, confirm) with a fake sessionId and the same phone.
2. Asserts a row appears in the `leads` table.
3. Cleans up.

**Expected output:**
Test passes in `npm test`.

## Stretch Goals (Optional - Extra Credit)

- Record every USSD session transcript in a `ussd_transcripts` table for auditing.
- Add a `created_via_session_id` column so you can trace a lead back to its USSD session.
- Add a stats endpoint `GET /api/stats/sources` returning counts per source.

## Submission Requirements

- **What to submit:** Repo, migration, updated dashboard, integration test, `AI_AUDIT.md`.
- **Deadline:** End of Day 3

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Source column added | 10 | Migration correct. Existing leads default to `manual`. |
| USSD save to Postgres | 30 | Real row appears after a USSD session. Bridge hand-written. |
| Source badge in dashboard | 15 | Visual distinction for the three sources. |
| Duplicate handling via upsert | 20 | No duplicate phones. |
| End-to-end integration test | 15 | Test posts callbacks and asserts DB row. Passes. |
| Clean commits | 10 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Using INSERT without handling conflict.** For upsert, use `ON CONFLICT (wa_phone) DO UPDATE SET ...`.
- **Forgetting to clear Redis after save.** A stale session confuses the next dial.
- **Swallowing errors silently.** At minimum, `console.error` so the operator can investigate.

## Resources

- Day 3 reading: [Connecting USSD to the CRM.md](./Connecting%20USSD%20to%20the%20CRM.md)
- Week 13 AI boundaries: [../ai.md](../ai.md)
