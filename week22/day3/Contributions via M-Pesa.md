# Week 22, Day 3: Contributions via M-Pesa

By the end of today, typing `/contribute` in the chama bot triggers a real M-Pesa STK Push to the member's phone, records a pending contribution, and upon callback confirms the contribution and announces it to the group. The full loop runs on the payments package from Week 19.

**Prior-week concepts you will use today:**
- The payments package (Week 19) -- `payments.initiate`, `payments.handleCallback`.
- Transactional outbox (Week 18, Day 2).
- Telegram inline keyboards (Week 20, Day 2).
- The chama schema (Week 22, Day 1).

**Estimated time:** 3 hours

---

## The Happy-Path Flow

Ten steps, as sketched in Day 1:

1. Member types `/contribute` in the bot.
2. Bot responds with a keyboard: `[Contribute KSh X for Month Y]` / `[Custom amount]` / `[Cancel]`.
3. Member clicks the preset.
4. Bot calls `chamaService.initiateContribution(memberUserId, chamaId, amountCents)`.
5. Service creates a `contributions` row with `status = 'pending'`.
6. Service calls `payments.initiate("mpesa", { orderId: contributionId, phone, amountCents })`.
7. Payments package initiates STK Push, returns `reference`.
8. Service updates the contribution row with `mpesa_reference`.
9. Bot replies to member: "Check your phone for the M-Pesa prompt. It expires in 60 seconds."
10. Webhook -> outbox -> Telegram announcement.

Each step is a few lines. The plumbing you already have.

---

## `/contribute` Handler

```javascript
// server/services/chama/commands/contribute.js
const chamaService = require("../chamaService");

module.exports = async function contribute(bot, message) {
  const chatId = message.chat.id;
  const userId = message.from.id;

  // Must be a group chat with a chama set up
  const chama = await chamaService.findByChatId(chatId);
  if (!chama) {
    return bot.sendMessage(chatId, "This group is not a chama. Run /setup first.");
  }
  const member = await chamaService.findMember(chatId, userId);
  if (!member) {
    return bot.sendMessage(chatId, "Run /join first.");
  }

  const cycle = await chamaService.getOpenCycle(chatId);
  if (!cycle) {
    return bot.sendMessage(chatId, "No open cycle. Ask the treasurer to run /open_cycle.");
  }

  const amountKsh = chama.monthly_amount_cents / 100;
  const periodLabel = new Date(cycle.period_start).toLocaleDateString("en-KE", { month: "long" });

  // Reply privately (do not spam the group with payment flows)
  await bot.sendMessage(userId,
    `Contribute for ${chama.name}:`,
    {
      reply_markup: {
        inline_keyboard: [
          [{ text: `Contribute KSh ${amountKsh} for ${periodLabel}`, callback_data: `cntb:${cycle.id}:${chama.monthly_amount_cents}` }],
          [{ text: "Custom amount", callback_data: `cntb:${cycle.id}:custom` }],
          [{ text: "Cancel", callback_data: "cntb:cancel" }],
        ],
      },
    }
  );
};
```

Notice the callback data shape: `cntb:<cycleId>:<amountCents>` or `cntb:<cycleId>:custom`. Packed tight to stay under the 64-byte limit.

---

## Handling The Callback

```javascript
// In the callback query handler:
if (data.startsWith("cntb:")) {
  const [, cycleId, amountPart] = data.split(":");
  if (amountPart === "cancel") {
    await bot.editMessageText("Cancelled.", { chat_id: query.message.chat.id, message_id: query.message.message_id });
    return bot.answerCallbackQuery(query.id);
  }
  if (amountPart === "custom") {
    await setSession(query.message.chat.id, query.from.id, {
      state: "awaiting_custom_amount",
      context: { cycleId: parseInt(cycleId, 10) },
    });
    await bot.sendMessage(query.message.chat.id, "Type the amount in KSh:");
    return bot.answerCallbackQuery(query.id);
  }

  const amountCents = parseInt(amountPart, 10);
  await bot.answerCallbackQuery(query.id);
  await initiateContribution(bot, query, parseInt(cycleId, 10), amountCents);
}
```

For the custom amount path, when the user types a number, a message handler sees the session state and calls `initiateContribution` with their number.

---

## `initiateContribution`

```javascript
async function initiateContribution(bot, query, cycleId, amountCents) {
  const userId = query.from.id;
  const chatId = query.message.chat.id;

  // Edit the original message to show processing state
  await bot.editMessageText("Initiating M-Pesa prompt...", {
    chat_id: chatId,
    message_id: query.message.message_id,
  });

  try {
    const result = await chamaService.initiateContribution({
      userId,
      cycleId,
      amountCents,
    });

    await bot.editMessageText(
      `Check your phone for the M-Pesa prompt.\n` +
      `Contribution id: ${result.contributionId.slice(0, 8)}\n` +
      `Enter your PIN to complete.`,
      { chat_id: chatId, message_id: query.message.message_id }
    );
  } catch (err) {
    console.error("Initiate contribution failed:", err);
    await bot.editMessageText(
      "Could not start M-Pesa payment. Please try again.",
      { chat_id: chatId, message_id: query.message.message_id }
    );
  }
}
```

---

## The Service Layer

```javascript
// server/services/chama/chamaService.js
const payments = require("mctaba-payments").init({ /* config */ });
const repo = require("./chamaRepo");

async function initiateContribution({ userId, cycleId, amountCents }) {
  // Load cycle and member
  const cycle = await repo.getCycle(cycleId);
  if (!cycle || cycle.status !== "open") throw new Error("Cycle not open");

  const chama = await repo.findByChatId(cycle.chama_id);
  const member = await repo.findMember(cycle.chama_id, userId);
  if (!member) throw new Error("Not a member");

  // Create contribution row
  const contribution = await repo.createContribution({
    cycleId,
    chamaId: cycle.chama_id,
    memberUserId: userId,
    amountCents,
    status: "pending",
  });

  // Initiate M-Pesa
  const init = await payments.initiate("mpesa", {
    orderId: contribution.id,
    phone: member.phone,
    amountCents,
  });

  await repo.updateContribution(contribution.id, {
    mpesa_reference: init.reference,
  });

  return { contributionId: contribution.id, reference: init.reference };
}
```

Clean separation: the command handler knows Telegram, the service knows business rules, the payments package knows M-Pesa, the repo knows SQL.

---

## The Callback Side

The payments package's `handleCallback` flips the contribution to `confirmed` on success. But it does not know about the chama -- it just sees "order `<uuid>` paid KSh X". How does the chama bot know to announce?

Answer: the outbox. When the payments package confirms a payment, it writes an `order.paid` event to the outbox. The outbox worker processes those events. You hook chama-specific processing into the worker:

```javascript
// server/workers/outbox.js (extended)
const chamaService = require("../services/chama/chamaService");
const telegramService = require("../services/telegram.service");

async function processOutboxRow(row) {
  if (row.event_type === "order.paid") {
    const orderId = row.payload.orderId;
    // Is this a chama contribution?
    const contribution = await query(
      "SELECT id, chama_id, member_user_id, amount_cents FROM contributions WHERE id = $1",
      [orderId]
    );
    if (contribution.rows[0]) {
      const c = contribution.rows[0];
      await chamaService.confirmContribution(c.id);
      await telegramService.sendMessage(c.chama_id,
        `${await memberName(c.chama_id, c.member_user_id)} just contributed KSh ${(c.amount_cents / 100).toLocaleString()}!`
      );
      // Also update the "cycle progress" status
      const stats = await chamaService.getGroupStats(c.chama_id);
      await telegramService.sendMessage(c.chama_id,
        `Cycle progress: KSh ${(stats.collectedThisCycle / 100).toLocaleString()} / ${(stats.expectedThisCycle / 100).toLocaleString()}`
      );
    } else {
      // Must be a shop order, not a chama contribution
      // ... existing shop confirmation logic
    }
  }
}
```

The outbox worker is a single switchboard: one event type, multiple consumers, each checking if the event is relevant to them.

---

## `confirmContribution`

```javascript
async function confirmContribution(contributionId) {
  // Atomic: only confirm if still pending
  const result = await query(
    `UPDATE contributions
     SET status = 'confirmed', confirmed_at = NOW()
     WHERE id = $1 AND status = 'pending'
     RETURNING id`,
    [contributionId]
  );
  return result.rowCount > 0;
}
```

The atomic update handles the "what if the callback fires twice?" case: the second call is a no-op.

---

## Handling Failed Contributions

When M-Pesa fails (user cancels, wrong PIN, timeout), the payments package marks the order `cancelled` or `failed`. An event goes to the outbox. The chama handler picks it up:

```javascript
if (row.event_type === "order.failed" || row.event_type === "order.cancelled") {
  const contribution = // look up
  if (contribution) {
    await query("UPDATE contributions SET status = 'failed' WHERE id = $1", [contribution.id]);
    await telegramService.sendMessage(contribution.member_user_id,
      "Your contribution did not go through. Try again with /contribute."
    );
    // Do NOT announce in the group -- failures are private
  }
}
```

Failed contributions are a private matter. Do not spam the group.

---

## Checkpoint

1. `/contribute` in a chama group replies privately with the three-button keyboard.
2. Clicking the preset fires a real STK Push to the test sandbox number.
3. Entering the PIN triggers the webhook, the contribution flips to `confirmed`, and the group announcement fires.
4. `/stats` shows the new total immediately.
5. Cancelling the STK flips the contribution to `failed` and sends a private "try again" DM, no group spam.
6. Running `/contribute` twice in quick succession does not create two rows for the same cycle slot -- enforce via optimistic UI.
7. The treasurer running `/members` sees the updated status.

Commit:

```bash
git add .
git commit -m "feat: chama contributions via mpesa with outbox-driven announcements"
```

---

## What You Learned

- Contributions reuse the payments package; the chama does not know about Daraja.
- The outbox worker is the switchboard between payments and chama.
- Success announcements are public; failure notifications are private.
- Atomic UPDATE patterns give you idempotency for free.

Tomorrow: cron jobs for end-of-cycle processing, fines, and reminders.
