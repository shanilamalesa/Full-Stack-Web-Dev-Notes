# Week 22 Weekend: Project 4 -- Chama Savings Platform

Ship the chama platform into a real group and run at least one contribution through it. By Monday morning you have screenshots of a real member contributing via M-Pesa and the bot announcing it.

**Estimated time:** 8-10 hours.

**Deadline:** Monday morning.

---

## Functional Requirements

### Setup
- Real Telegram group with at least 3 members (you + 2 friends or family).
- `/setup` configures the chama with a name, monthly amount, and cycle day.
- All 3+ members successfully `/join` and provide their M-Pesa phone numbers.

### Contributions
- At least 1 member (ideally 2-3) contributes via M-Pesa sandbox end-to-end.
- Contribution announcement fires in the group chat.
- `/balance` and `/stats` reflect the contribution accurately.

### Treasurer experience
- `/dashboard` link works and opens the web dashboard.
- Treasurer can see member list, cycle progress, and outstanding fines.
- Treasurer can manually close a cycle from the dashboard.

### Cron
- The reminder cron fires at the expected time (test by setting the cron temporarily to run in the next few minutes).
- End-of-cycle cron closes the cycle, writes fines, opens a new one, and posts the summary.

---

## Technical Requirements

1. All chama code lives under `server/services/chama/`.
2. Uses `mctaba-payments` package from the registry.
3. Transactions around every cycle-close operation.
4. Idempotent contribution callbacks.
5. Rate-limited Telegram sends (20/sec cap).
6. Seed script `db/chama-seed.sql` creates one test chama with 3 members.
7. `README.md` includes: the group story, screenshots, commands, and how to test end-to-end.

---

## Deliverables

- [ ] Working bot in a real Telegram group.
- [ ] Screenshot of a member typing /contribute.
- [ ] Screenshot of the M-Pesa prompt arriving.
- [ ] Screenshot of the bot's "contribution confirmed" announcement.
- [ ] Screenshot of the treasurer dashboard showing the contribution.
- [ ] Screenshot of the cycle-closed summary posted in the group.
- [ ] README with the group story in one paragraph.

---

## Grading Rubric (100 pts)

| Area | Points |
|---|---|
| Real group with real members | 10 |
| /setup, /join, /balance, /stats, /members, /contribute all work | 25 |
| End-to-end M-Pesa contribution with screenshot | 20 |
| Outbox-driven announcements | 10 |
| Cycle cron closes and opens correctly | 10 |
| Reminder cron fires with group + private DMs | 5 |
| Treasurer dashboard accessible via /dashboard | 10 |
| Invariants respected (check with pathological inputs) | 5 |
| README quality | 5 |

90+ is an A. 70+ is a pass.

---

## Hints

**Pick a real friend group.** Your Saturday cricket club, your college friends, your family group chat. Tell them you are building something for them. They will help test and they will tell you what is missing.

**Sandbox is fine for the demo.** The M-Pesa sandbox accepts test payments. You do not need live Daraja credentials for the grading demo. If your real friends test with real phones, use the sandbox and explain in the demo that production swaps the environment.

**Do not try to add payouts.** Payouts (money leaving the chama) is genuinely hard because it requires Daraja B2C with float, not just STK Push. Defer to a stretch goal.

**Kiswahili is a bonus.** If you have time, translate the key commands. A chama in Makueni or Kisii will use the Kiswahili version first.

**The treasurer's UX matters more than the members'.** Members only talk to the bot once a month. The treasurer uses the dashboard every week. Polish the dashboard over the member flows if you run out of time.

Good luck. Next week we turn queuing into a real discipline with BullMQ.
