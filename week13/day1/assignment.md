# Week 13 - Day 1 Assignment

## Title
First USSD Menu Via Africa's Talking Sandbox

## Overview
Week 13 brings USSD -- the channel that reaches every feature phone in Kenya. Today you register an Africa's Talking sandbox account, expose an Express endpoint that answers with `CON` and `END` responses, and dial a real code from a simulator to see your own menu.

## Learning Objectives Assessed
- Register for Africa's Talking sandbox and create a USSD channel
- Understand `CON` (continue) and `END` (terminate) response prefixes
- Read the four fields Africa's Talking sends on every callback
- Respond to a USSD request within the character budget

## Prerequisites
- Week 12 completed
- Your Express backend running with ngrok

## AI Usage Rules

**Ratio this week:** 45% manual / 55% AI
**Habit:** Protocol first, code second -- read the real docs before prompting. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining a quirk of AT's callback format after you read the docs.
- **NOT ALLOWED FOR:** Generating the first USSD handler or pasting `CON`/`END` logic without reading.
- **AUDIT REQUIRED:** Yes. Include a "Protocol quirks log" with at least 5 quirks learned from AT's docs.

## Tasks

### Task 1: Register AT sandbox

**What to do:**
1. Sign up at https://account.africastalking.com/
2. Switch to Sandbox mode.
3. Create a USSD channel. Pick a shortcode like `*384*1234#`.
4. Set the callback URL to `https://YOUR_NGROK/ussd` (for now, just a placeholder -- we will wire it next).
5. Note your username (`sandbox`) and API key.

**Expected output:**
USSD channel registered. Screenshot `day1-at-channel.png`.

### Task 2: Write the USSD handler by hand

**What to do:**
Create `packages/backend/src/routes/ussd.js`:

```javascript
const express = require("express");
const router = express.Router();

router.post("/", express.urlencoded({ extended: false }), (req, res) => {
  const { sessionId, serviceCode, phoneNumber, text } = req.body;

  console.log({ sessionId, serviceCode, phoneNumber, text });

  let response = "";
  if (text === "") {
    response = "CON Welcome to Mctaba CRM\n1. My leads\n2. New lead\n3. Exit";
  } else if (text === "1") {
    response = "END Your leads feature is coming soon.";
  } else if (text === "2") {
    response = "END New lead feature is coming soon.";
  } else if (text === "3") {
    response = "END Asante. Bye.";
  } else {
    response = "END Invalid option.";
  }

  res.set("Content-Type", "text/plain");
  res.send(response);
});

module.exports = router;
```

Mount at `/ussd`. Every line hand-typed.

**Expected output:**
POST /ussd responds with the menu.

### Task 3: Test with AT simulator

**What to do:**
1. In AT dashboard, open the USSD simulator.
2. Dial your shortcode.
3. Walk through each menu option.
4. Screenshot the full flow as `day1-simulator.png`.

**Expected output:**
Simulator shows your menu. Your Express logs show the incoming callbacks.

### Task 4: Handle the 182-character budget

**What to do:**
USSD screens are limited to roughly 182 characters per response. Write a small helper:

```javascript
function clamp(text, max = 180) {
  if (text.length <= max) return text;
  return text.slice(0, max - 3) + "...";
}
```

Apply it to every response. Try a deliberately long menu to see it truncate.

**Expected output:**
No response exceeds 182 characters. Long responses are safely clamped.

### Task 5: Protocol quirks log

**What to do:**
Read Africa's Talking's USSD documentation. In `AI_AUDIT.md`, add a "Protocol quirks log" with at least 5 quirks you learned:

```markdown
## Protocol quirks log (from AT docs)
1. Every response starts with CON or END.
2. Sessions expire after ~180 seconds.
3. The `text` field is cumulative (e.g. `1*2*3`), not the latest input.
4. Character budget is ~182 per screen.
5. Invalid menu options should re-render with CON, never END.
```

Every quirk must come from the docs, not AI paraphrasing.

**Expected output:**
Audit updated.

## Stretch Goals (Optional - Extra Credit)

- Log every USSD callback to a file for later debugging.
- Add a simple "back" option that returns to the main menu.
- Dial from a real phone (if your sandbox supports it) in addition to the simulator.

## Submission Requirements

- **What to submit:** Repo with ussd.js route, screenshots, `AI_AUDIT.md`.
- **Deadline:** End of Day 1

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| AT sandbox USSD channel created | 15 | Screenshot confirms. |
| USSD handler by hand | 30 | CON/END responses correct. No AI in file. |
| Simulator flow works | 15 | Screenshot shows the menu from the simulator. |
| 182-character clamp | 10 | Helper applied. |
| Protocol quirks log | 25 | At least 5 quirks from the real AT docs. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Returning JSON instead of text.** USSD expects plain text. Set `Content-Type: text/plain`.
- **Using `END` where `CON` is needed.** `END` terminates the session. If the user typed a wrong option, re-render the menu with `CON`.
- **Exceeding the character budget.** Some phones crop silently. Always clamp.

## Resources

- Day 1 reading: [USSD Foundations.md](./USSD%20Foundations.md)
- Week 13 AI boundaries: [../ai.md](../ai.md)
- Africa's Talking USSD: https://developers.africastalking.com/docs/ussd/overview
