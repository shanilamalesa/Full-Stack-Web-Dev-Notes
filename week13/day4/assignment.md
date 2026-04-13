# Week 13 - Day 4 Assignment

## Title
USSD Polish -- i18n, Error Handling, and Feature-Phone UX

## Overview
Day 4 is pre-weekend polish day. Today you add Kiswahili translations, handle every error path gracefully, cap every response at the character budget, and stress-test the UX from a user's perspective -- a farmer in Bungoma on a Nokia 105.

## Learning Objectives Assessed
- Build a small i18n helper (`t(lang, key)`)
- Add Kiswahili strings for every screen
- Handle errors with friendly fallbacks
- Compress messages to fit the 182-character budget
- Pass a ship checklist

## Prerequisites
- Days 1-3 completed

## AI Usage Rules

**Ratio:** 45/55. **Habit:** Protocol first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Swahili translation suggestions (have a native-speaker review them).
- **NOT ALLOWED FOR:** Deciding which strings to translate.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: i18n helper

**What to do:**
Create `src/i18n/messages.js`:

```javascript
const messages = {
  en: {
    welcome: "Welcome",
    invalid: "Invalid option",
    goodbye: "Goodbye",
    menu: "1. Add lead\n2. Exit",
    enterName: "Enter lead name:",
    enterPhone: "Enter lead phone:",
    confirmTemplate: (name, phone) => `Confirm:\nName: ${name}\nPhone: ${phone}\n1. Save\n2. Cancel`,
    saved: "Lead saved. Asante.",
    cancelled: "Cancelled.",
  },
  sw: {
    welcome: "Karibu",
    invalid: "Chaguo batili",
    goodbye: "Kwaheri",
    menu: "1. Ongeza lead\n2. Ondoka",
    enterName: "Andika jina la lead:",
    enterPhone: "Andika nambari ya simu:",
    confirmTemplate: (name, phone) => `Thibitisha:\nJina: ${name}\nSimu: ${phone}\n1. Hifadhi\n2. Futa`,
    saved: "Lead imehifadhiwa. Asante.",
    cancelled: "Imefutwa.",
  },
};

function t(lang, key, ...args) {
  const bundle = messages[lang] || messages.en;
  const value = bundle[key];
  return typeof value === "function" ? value(...args) : value;
}

module.exports = { t };
```

**Expected output:**
Helper works. `t("sw", "welcome")` returns "Karibu".

### Task 2: Language toggle on first menu

**What to do:**
Add a language selector as the first screen:

```
CON Language / Lugha
1. English
2. Kiswahili
```

Store the choice in the session data. Every subsequent response uses `t(session.data.lang, ...)`.

**Expected output:**
Choosing language at the start changes every screen's language.

### Task 3: Error handling

**What to do:**
Wrap your main handler in a try/catch. Any unhandled error returns a friendly message:

```javascript
router.post("/", async (req, res) => {
  try {
    const response = await handleUssd(req.body);
    res.set("Content-Type", "text/plain").send(response);
  } catch (err) {
    console.error("USSD handler error:", err);
    res.set("Content-Type", "text/plain").send("END Service temporarily unavailable. Try again soon.");
  }
});
```

Deliberately throw an error inside one state to test the fallback.

**Expected output:**
Errors return a friendly `END` message, not a 500.

### Task 4: Pre-weekend checklist

**What to do:**
Create `CHECKLIST.md`:

```markdown
## Week 13 Day 4 Pre-Weekend Checklist

- [ ] AT sandbox USSD channel live and reachable
- [ ] State machine handles welcome, name, phone, confirm
- [ ] Redis TTL cleans up sessions
- [ ] Leads save to Postgres with source='ussd'
- [ ] No duplicates via upsert
- [ ] i18n helper with English and Kiswahili
- [ ] Language toggle works
- [ ] Error fallback returns friendly END message
- [ ] Every response under 182 chars
- [ ] AI_AUDIT.md has protocol quirks log current
```

Tick honestly.

**Expected output:**
`CHECKLIST.md` committed.

### Task 5: UX audit from a user's perspective

**What to do:**
In `ux-audit.md`, imagine you are a farmer on a Nokia 105 without a smartphone. Walk through your flow 3 times. Write 5-7 observations on what feels slow, confusing, or risky. Then list 3 improvements you would make.

No AI. Your own imagination.

**Expected output:**
`ux-audit.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Add a third language (French, Arabic, or any cohort mate's).
- Add a "help" command accessible from every screen.
- Track session abandon rate by logging terminations to a `ussd_drops` table.

## Submission Requirements

- **What to submit:** Repo, `i18n/messages.js`, `CHECKLIST.md`, `ux-audit.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 4

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| i18n helper with two languages | 20 | Works with `t()` function. |
| Language toggle at start | 15 | Changes all subsequent screens. |
| Error handler with friendly fallback | 20 | Deliberate throw returns friendly END. |
| Character budget enforced | 10 | Every response under 182. |
| Pre-weekend checklist | 10 | Honest. |
| UX audit notes | 20 | Five observations plus three improvements. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Translating everything to AI-Swahili without review.** AI Swahili is often too formal or machine-sounding. Ask a native speaker to sanity-check.
- **Hardcoding strings in multiple places.** Always route through the `t()` helper.
- **Letting a 500 reach the user.** Wrap everything.

## Resources

- Day 4 reading: [Polishing for Feature Phones.md](./Polishing%20for%20Feature%20Phones.md)
- Week 13 AI boundaries: [../ai.md](../ai.md)
