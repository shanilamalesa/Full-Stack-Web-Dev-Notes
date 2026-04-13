# Week 16 - AI Boundaries

**Ratio this week: 30% Manual / 70% AI**
**Habit introduced: "AI as integration engineer."**
**Shift from last week: Another 5% AI. Project 3 is the first full-stack project where AI can realistically do most of the typing.**

Project 3 -- E-commerce Lite + WhatsApp Updates -- is the week where everything you learned in Weeks 10-15 snaps together. Cart and checkout you built last week. M-Pesa you built in Week 10. WhatsApp bot you built in Week 11. Dashboard patterns you have built many times. AI handles 70% of this week because most of it is integration of known parts.

The habit is the new one: treat AI like a junior integration engineer working under your direction. You tell it what to wire to what. It runs the wiring. You verify the connections.

---

## Why The Ratio Moved

You are literally re-using last week's cart, Week 14's catalogue, Week 10's M-Pesa flow, and Week 11's WhatsApp bot. The only new work is gluing them together and adding Project 3 polish (order confirmations, admin dashboard, retries). AI excels at gluing. Let it glue.

The **money rule** from Week 10 still holds. The **auth rule** from Week 12 still holds. The "never trust AI with money" bucket covers every line that touches a transaction amount, a reference, or an idempotency key.

---

## What You MUST Do Manually (30%)

### Day 1 -- M-Pesa inside the shop
- Wire up your Week 10 M-Pesa flow to the Next.js shop's Server Actions.
- Every amount-validation line is manual. Every reference check is manual. Every idempotency key is manual.
- The actual Server Action plumbing (receiving the form, calling the service, redirecting) can be AI-assisted after you define the function signature.

### Day 2 -- Handling the M-Pesa callback
- Callback handler is manual (money rule).
- The UI that displays "payment pending" / "paid" / "failed" can be AI-assisted.
- Test the full callback path with sandbox numbers.

### Day 3 -- WhatsApp order confirmations
- Reuse the Week 11 bot to send a confirmation message when an order moves to `paid`.
- You write the template and the trigger point manually. AI can help with the message formatter.

### Day 4 -- Notifications, retries, polish
- Retries for failed notifications -- the retry logic is manual (edge cases matter). The dashboard showing retry status can be AI.

### Day 5 and weekend -- Project 3 ship
- Full end-to-end test: customer checks out, pays, gets a WhatsApp confirmation, owner sees the order in the dashboard.

---

## What You CAN Use AI For (70%)

- **Wiring:** connecting a Server Action to a service, connecting a callback to a notification, connecting the admin to the orders table.
- **UI scaffolding:** everything visual.
- **Test generation:** after you have written the core paths manually.
- **Error messages:** writing user-facing copy.

Forbidden (same as Week 10):
- Writing amount validation.
- Writing idempotency checks.
- Writing signature verification.
- Deciding when to mark an order paid.

---

## Things AI Is Bad At This Week

- **Cross-service state.** "The order says paid but the notification did not fire" is a distributed bug. AI rarely catches these from a single file.
- **Real-world race conditions.** Two callbacks arriving at once. Two customers buying the last unit. AI defaults to happy paths.
- **Retry policies.** Exponential backoff is easy to write wrong. Specify explicitly.

---

## Core Mental Models For This Week

- **An order is a state machine.** `pending -> paid -> fulfilled -> shipped` or `pending -> failed`. Every transition has a guard.
- **A callback is the same order progressing through the machine.** External events drive the state forward.
- **A notification is a side effect of a transition, not the transition itself.** Keep them separate.

---

## This Week's AI Audit Focus

Add a **wiring diagram** to your audit showing which services are connected to which, with arrows labelled by the function call or event type.

---

## Assessment

- End-to-end live demo: a real order, a real payment, a real WhatsApp confirmation.
- Facilitator asks: "what happens if the user's phone dies between STK prompt and PIN entry?" You walk the exact code path.
- Show your wiring diagram and defend every arrow.

---

## One Sentence To Remember

"Your job this week is wiring. AI's job is the wires. Keep the diagram in your head and review the connections."
