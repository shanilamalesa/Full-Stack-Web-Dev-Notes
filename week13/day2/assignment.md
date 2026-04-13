# Week 13 - Day 2 Assignment

## Title
USSD State Machine With Redis Sessions

## Overview
Yesterday's menu was stateless. Today you add a real per-user state machine backed by Redis. Users can navigate deeper than one level, enter free text, and have their progress remembered between hops.

## Learning Objectives Assessed
- Install and connect to Redis from Node
- Use Redis with an expiring key to store USSD session state
- Implement a state machine that advances based on `text` input
- Keep each handler small and testable

## Prerequisites
- Day 1 completed

## AI Usage Rules

**Ratio:** 45/55. **Habit:** Protocol first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Explaining Redis TTL behaviour after you set one.
- **NOT ALLOWED FOR:** Generating the state machine structure.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Install Redis and connect

**What to do:**
- macOS: `brew install redis && brew services start redis`
- Ubuntu: `sudo apt install redis-server && sudo systemctl start redis-server`
- Docker: `docker run -d --name redis -p 6379:6379 redis:7`

Install the client:

```bash
npm install ioredis
```

Create `src/db/redis.js`:

```javascript
const Redis = require("ioredis");
const redis = new Redis(process.env.REDIS_URL || "redis://localhost:6379");
redis.on("error", (err) => console.error("Redis error:", err));
module.exports = redis;
```

**Expected output:**
Node script `node -e 'require("./src/db/redis").ping().then(console.log)'` prints `PONG`.

### Task 2: State machine with four states

**What to do:**
Design the states:
- `WELCOME` -- first screen, menu
- `AWAITING_NAME` -- user is entering a lead name
- `AWAITING_PHONE` -- user is entering a phone
- `CONFIRM` -- preview and confirm

State lives in Redis with a 3-minute TTL:

```javascript
const KEY = (sessionId) => `ussd:session:${sessionId}`;

async function getState(sessionId) {
  const raw = await redis.get(KEY(sessionId));
  if (!raw) return { state: "WELCOME", data: {} };
  return JSON.parse(raw);
}

async function setState(sessionId, session) {
  await redis.set(KEY(sessionId), JSON.stringify(session), "EX", 180);
}
```

Write the handler:

```javascript
async function handleUssd({ sessionId, text }) {
  const session = await getState(sessionId);
  const input = text.split("*").pop() || "";

  if (session.state === "WELCOME") {
    if (input === "") return "CON Welcome\n1. Add lead\n2. Exit";
    if (input === "1") {
      session.state = "AWAITING_NAME";
      await setState(sessionId, session);
      return "CON Enter lead name:";
    }
    if (input === "2") return "END Goodbye";
    return "CON Invalid. 1. Add lead 2. Exit";
  }

  if (session.state === "AWAITING_NAME") {
    session.data.name = input;
    session.state = "AWAITING_PHONE";
    await setState(sessionId, session);
    return "CON Enter lead phone:";
  }

  if (session.state === "AWAITING_PHONE") {
    session.data.phone = input;
    session.state = "CONFIRM";
    await setState(sessionId, session);
    return `CON Confirm:\nName: ${session.data.name}\nPhone: ${session.data.phone}\n1. Save\n2. Cancel`;
  }

  if (session.state === "CONFIRM") {
    if (input === "1") {
      // TODO Day 3: save to Postgres
      await redis.del(KEY(sessionId));
      return "END Lead saved. Asante.";
    }
    if (input === "2") {
      await redis.del(KEY(sessionId));
      return "END Cancelled.";
    }
    return "CON Invalid. 1. Save 2. Cancel";
  }

  return "END Session error.";
}
```

Type it yourself.

**Expected output:**
A multi-step flow works end-to-end in the simulator.

### Task 3: Split `text` correctly

**What to do:**
Understand that `text` accumulates: on the third hop it might be `1*John*0712345678`. Your handler must read only the latest segment (`text.split("*").pop()`). Verify this in your logs.

**Expected output:**
Logs show the raw `text` field and your parsed input on each hop.

### Task 4: Input validation

**What to do:**
Reject invalid phone numbers (length, digits) by re-rendering the same menu with `CON` and an error prefix:

```javascript
if (!/^0\d{9}$/.test(input)) {
  return `CON Invalid phone. Enter lead phone (e.g. 0712345678):`;
}
```

**Expected output:**
Bad phone numbers bounce back; valid ones advance.

### Task 5: Trace a full session

**What to do:**
Walk through a full happy-path session in the simulator. Screenshot each step (or one combined screenshot) as `day2-session.png`. In `session-trace.md`, describe what is in Redis at each step of the flow.

**Expected output:**
`session-trace.md` committed.

## Stretch Goals (Optional - Extra Credit)

- Add a "back" option (usually `0`) that reverses one state.
- Add a max-session timer that terminates long flows.
- Add a test that simulates POSTs without touching a real AT simulator.

## Submission Requirements

- **What to submit:** Repo, state machine, `session-trace.md`, `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Redis running and connected | 10 | PING works from Node. |
| State machine with 4 states | 30 | All transitions work in the simulator. |
| Correct text parsing | 15 | Latest segment extracted; full history not confused. |
| Input validation | 15 | Bad input re-prompts with CON. |
| Session trace notes | 20 | Student explains Redis content at each step in their own words. |
| Clean commits | 10 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Reading the full `text` as one answer.** Always split and take the last segment.
- **Forgetting TTL on Redis keys.** Without TTL, stale sessions live forever. 180s is the AT session lifetime.
- **Mutating state after reading.** Always `await setState()` after changing `session`.

## Resources

- Day 2 reading: [State Machines and Redis Sessions.md](./State%20Machines%20and%20Redis%20Sessions.md)
- Week 13 AI boundaries: [../ai.md](../ai.md)
- ioredis docs: https://github.com/redis/ioredis
