# Week 22, Day 2: Chama Bot Core

By the end of today, the Telegram bot has the chama-specific commands wired: `/setup` to turn a group into a chama, `/join` for members, `/balance` for a member's own state, `/stats` for the group summary, and `/members` for the treasurer. Contributions (Day 3) come next.

**Prior-week concepts you will use today:**
- Telegram bot with inline keyboards (Week 20)
- The chama schema from yesterday.

**Estimated time:** 3-4 hours

---

## Which File Lives Where

Add a new folder `server/services/chama/` to keep the chama-specific code separate from generic Telegram code:

```
server/services/
  chama/
    chamaService.js        # business logic (pure)
    chamaRepo.js           # SQL
    commands/
      setup.js
      join.js
      balance.js
      stats.js
      members.js
    cycleService.js        # end-of-cycle logic (Day 4)
  telegram.service.js      # generic Telegram send/receive
```

Each command file exports a single function that takes `(bot, message, context)` and runs one command. The telegram handler dispatches to them.

---

## `/setup` Command

```javascript
// server/services/chama/commands/setup.js
const chamaService = require("../chamaService");

module.exports = async function setup(bot, message, args) {
  const chatId = message.chat.id;

  if (message.chat.type === "private") {
    await bot.sendMessage(chatId, "Setup must be run inside the chama group chat.");
    return;
  }

  const existing = await chamaService.findByChatId(chatId);
  if (existing) {
    await bot.sendMessage(chatId, `This group is already set up as "${existing.name}".`);
    return;
  }

  // Only group admins can set up
  const member = await bot.getChatMember(chatId, message.from.id);
  if (member.status !== "creator" && member.status !== "administrator") {
    await bot.sendMessage(chatId, "Only a group admin can run /setup.");
    return;
  }

  // Expected args: name amount_ksh cycle_day
  if (args.length < 3) {
    await bot.sendMessage(chatId,
      "Usage: /setup <name> <monthly_amount_ksh> <cycle_day>\n" +
      "Example: /setup Kilimani Chama 1000 1"
    );
    return;
  }

  const name = args.slice(0, args.length - 2).join(" ");
  const monthlyKsh = parseInt(args[args.length - 2], 10);
  const cycleDay = parseInt(args[args.length - 1], 10);

  if (isNaN(monthlyKsh) || monthlyKsh <= 0) {
    await bot.sendMessage(chatId, "Invalid monthly amount.");
    return;
  }
  if (isNaN(cycleDay) || cycleDay < 1 || cycleDay > 28) {
    await bot.sendMessage(chatId, "Cycle day must be between 1 and 28.");
    return;
  }

  await chamaService.create({
    chatId,
    name,
    monthlyAmountCents: monthlyKsh * 100,
    cycleDay,
    treasurerUserId: message.from.id,
  });

  await bot.sendMessage(chatId,
    `"${name}" is now a chama.\n` +
    `Monthly: KSh ${monthlyKsh}\n` +
    `Cycle day: ${cycleDay}\n` +
    `Treasurer: ${message.from.first_name}\n\n` +
    `Members: type /join to register.`
  );
};
```

Design choices:

**Text parsing, not keyboards, for setup.** Setup happens once. A single typed command is faster than a multi-step menu. Members use menus; admins use commands.

**Treasurer is whoever runs `/setup`.** Simple. Change it later with another command.

**cycle_day 1-28.** Not 29-31 because February exists. Cron fires on day N; if a month has no day 29, the cycle is late by a day. 1-28 avoids this forever.

---

## `/join` Command

```javascript
// server/services/chama/commands/join.js
const chamaService = require("../chamaService");

module.exports = async function join(bot, message) {
  const chatId = message.chat.id;
  const userId = message.from.id;
  const name = message.from.first_name || "Member";

  const chama = await chamaService.findByChatId(chatId);
  if (!chama) {
    await bot.sendMessage(chatId, "This group is not set up as a chama. Ask an admin to run /setup.");
    return;
  }

  const existing = await chamaService.findMember(chatId, userId);
  if (existing && !existing.left_at) {
    await bot.sendMessage(chatId, `${name}, you are already a member.`);
    return;
  }

  // We need the member's phone to run M-Pesa on them. Ask privately.
  await bot.sendMessage(userId,
    `Welcome to ${chama.name}. Please reply with your M-Pesa phone number in the format +254712345678.`
  ).catch(async (err) => {
    if (err.response?.body?.error_code === 403) {
      await bot.sendMessage(chatId,
        `${name}, please start a private chat with me first, then run /join again.`
      );
    }
  });

  // Set bot state to "collecting phone number for this user"
  await chamaService.startMemberOnboarding(userId, chatId);
};
```

Joining is a two-step flow: issue the command in the group, finish in a private chat so the phone number is not pasted in public.

When the member replies with their phone number in a private chat:

```javascript
// In handleMessage, before normal command parsing:
const onboarding = await chamaService.getMemberOnboarding(userId);
if (onboarding && /^\+?\d{9,14}$/.test(text)) {
  await chamaService.completeMemberOnboarding({
    userId,
    chamaId: onboarding.chamaId,
    name: message.from.first_name,
    phone: text,
  });
  await bot.sendMessage(userId, `You are now a member. Contribute anytime with /contribute.`);
  // Announce in the group
  await bot.sendMessage(onboarding.chamaId, `Welcome ${message.from.first_name}! Total members: ${newCount}`);
  return;
}
```

---

## `/balance` Command

```javascript
// server/services/chama/commands/balance.js
module.exports = async function balance(bot, message) {
  const chatId = message.chat.id;
  const userId = message.from.id;

  // Find the chama -- if in a group, use this chat; if private, look up which chama the user is in.
  let chamaId;
  if (message.chat.type === "private") {
    const memberships = await chamaService.findMemberships(userId);
    if (memberships.length === 0) {
      return bot.sendMessage(chatId, "You are not a member of any chama.");
    }
    if (memberships.length > 1) {
      return bot.sendMessage(chatId, "You are in multiple chamas. Run /balance from inside the specific group chat.");
    }
    chamaId = memberships[0].chama_id;
  } else {
    chamaId = chatId;
  }

  const data = await chamaService.getMemberBalance(chamaId, userId);
  if (!data) {
    return bot.sendMessage(chatId, "You are not a member of this chama.");
  }

  const text =
    `${data.name}, your summary:\n` +
    `Cycle this month: KSh ${(data.contributedThisCycle / 100).toLocaleString()} / ${(data.expected / 100).toLocaleString()}\n` +
    `Total contributed (all time): KSh ${(data.totalContributed / 100).toLocaleString()}\n` +
    `Outstanding fines: KSh ${(data.outstandingFines / 100).toLocaleString()}`;

  await bot.sendMessage(userId, text); // private
};
```

Note: balance responses go to the private chat, not the group. Privacy.

---

## `/stats` Command (Group-Wide)

```javascript
module.exports = async function stats(bot, message) {
  const chatId = message.chat.id;
  const chama = await chamaService.findByChatId(chatId);
  if (!chama) return;

  const data = await chamaService.getGroupStats(chatId);

  const text =
    `${chama.name} - This cycle\n` +
    `Collected: KSh ${(data.collectedThisCycle / 100).toLocaleString()} / ${(data.expectedThisCycle / 100).toLocaleString()}\n` +
    `Contributors: ${data.contributors} / ${data.totalMembers}\n` +
    `Outstanding fines: KSh ${(data.outstandingFines / 100).toLocaleString()}\n` +
    `All-time total: KSh ${(data.allTime / 100).toLocaleString()}`;

  await bot.sendMessage(chatId, text);
};
```

Public. Everyone in the group sees it. This is the "progress bar" the chama watches during the month.

---

## `/members` Command (Treasurer-only)

```javascript
module.exports = async function members(bot, message) {
  const chatId = message.chat.id;
  const userId = message.from.id;

  const chama = await chamaService.findByChatId(chatId);
  if (!chama || chama.treasurer_user_id !== userId) {
    return bot.sendMessage(chatId, "Only the treasurer can run /members.");
  }

  const list = await chamaService.getMemberStatusList(chatId);
  const lines = list.map((m) => {
    const icon = m.contributedThisCycle >= chama.monthly_amount_cents ? "OK" : "MISSING";
    return `${icon} ${m.name}: KSh ${(m.contributedThisCycle / 100).toLocaleString()}`;
  });

  // Reply in private to the treasurer, not in the group.
  await bot.sendMessage(userId, `Members status:\n${lines.join("\n")}`);
};
```

The treasurer gets the full list privately and can nudge anyone who has not paid.

---

## Service And Repo Skeletons

The commands are thin. Most of the weight lives in the service and the repo. Outline:

```javascript
// server/services/chama/chamaService.js
const repo = require("./chamaRepo");

async function create({ chatId, name, monthlyAmountCents, cycleDay, treasurerUserId }) {
  // Also opens the first cycle
  return repo.create({ chatId, name, monthlyAmountCents, cycleDay, treasurerUserId });
}

async function findByChatId(chatId) { return repo.findByChatId(chatId); }
async function findMember(chatId, userId) { return repo.findMember(chatId, userId); }
async function getMemberBalance(chatId, userId) {
  const member = await repo.findMember(chatId, userId);
  const totals = await repo.getMemberTotals(chatId, userId);
  return { ...member, ...totals };
}
async function getGroupStats(chatId) { return repo.getGroupStats(chatId); }
async function getMemberStatusList(chatId) { return repo.getMemberStatusList(chatId); }
// ...

module.exports = { create, findByChatId, findMember, getMemberBalance, getGroupStats, getMemberStatusList /* ... */ };
```

Each function in `chamaRepo.js` is a short SQL query against the schema from Day 1.

Write one or two of the repo functions fully to internalise the shape. The rest follow the same pattern.

---

## Checkpoint

1. `/setup Kilimani 1000 1` in a group you are admin of creates a chama row and announces it.
2. `/join` sent in the group prompts the user to reply privately.
3. Replying in private with a phone number completes onboarding and announces the new member in the group.
4. `/balance` in private shows the user their status for the only chama they are in.
5. `/stats` in the group shows the public summary.
6. `/members` by the treasurer shows the status list privately.
7. Non-treasurer running `/members` gets "only the treasurer".

Commit:

```bash
git add .
git commit -m "feat: chama bot commands setup join balance stats members"
```

---

## What You Learned

- Commands are thin; services and repos hold the logic.
- Public vs private replies matter for UX and privacy.
- Two-step flows (group command + private reply) protect sensitive data.
- Validation belongs in the command handler, not the service.

Tomorrow we wire real contributions through the payments package.
