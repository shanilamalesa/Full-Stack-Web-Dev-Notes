# Mctaba Labs -- Fullstack Web Dev Curriculum

## AI Integration Map: Manual/AI Boundaries by Week

This document is the master reference for how AI is used across the 30-week programme. Every week has a per-week `ai.md` file with full detail; this file is the overview, the progression, and the permanent rules.

---

## How To Read This Document

Each week defines three things:

1. **Manual Zone** -- what students MUST write by hand. No AI. This is where the new learning happens.
2. **AI-Assisted Zone** -- what students CAN use AI for, but must submit an AI Audit (prompt used, changes made, what they learned).
3. **AI Ratio** -- the approximate split between manual and AI-assisted work that week.

The ratio shifts from ~90% manual in Week 1 to ~20% manual by Week 30. By graduation, students operate like working engineers: AI-assisted but engineer-driven.

For the full detail of any week, read that week's `ai.md` (e.g., [`week1/ai.md`](./week1/ai.md)).

---

## The Weekly Progression Table

| Week | Topic | Ratio (Manual / AI) | Habit Introduced |
|------|-------|---------------------|------------------|
| 1    | Environment setup + HTML | 90 / 10 | Ask AI to explain, not to do |
| 2    | CSS fundamentals | 85 / 15 | Compare and contrast |
| 3    | JavaScript basics | 80 / 20 | The 15-minute rule |
| 4    | JS functions + DOM | 75 / 25 | Build first, enhance with AI |
| 5    | JS OOP + modules | 70 / 30 | AI as code reviewer |
| 6    | Async + Fetch | 65 / 35 | Read the docs before the prompt |
| 7    | React foundations | 60 / 40 | New framework, manual first |
| 8    | React forms, routing, effects | 55 / 45 | AI as pair partner |
| 9    | React APIs, state, custom hooks, testing | 50 / 50 | AI reads docs, you make decisions |
| 10   | M-Pesa payments | 50 / 50 | **Never trust AI with money** (permanent rule) |
| 11   | WhatsApp Cloud API + CRM | 45 / 55 | New tech = manual. Repeated patterns = AI |
| 12   | Postgres + JWT auth | 45 / 55 | **Never trust AI with auth** (permanent rule) |
| 13   | USSD + Redis + Africa's Talking | 45 / 55 | Protocol first, code second |
| 14   | Next.js 14 App Router | 40 / 60 | Framework docs before framework prompts |
| 15   | Cart + checkout state machine | 35 / 65 | Model the state machine on paper |
| 16   | Project 3: E-commerce + WhatsApp | 30 / 70 | AI as integration engineer |
| 17   | Stripe + Airtel + unified payments | 30 / 70 | Provider docs first, implementation second |
| 18   | Webhook security, idempotency, refunds | 45 / 55 | Audit every security line (**SECURITY DIP**) |
| 19   | Payments package extraction | 35 / 65 | You design the API, AI fills it in |
| 20   | Telegram bots | 40 / 60 | New channel, manual handshake |
| 21   | Cron jobs and scheduled tasks | 30 / 70 | AI writes schedules, you decide on reliability |
| 22   | Project 4: Chama Savings Platform | 25 / 75 | Model the domain yourself |
| 23   | BullMQ queues + workers | 35 / 65 | Prototype alone, productionize with AI |
| 24   | Microservices split | 45 / 55 | Draw the split on paper (**ARCHITECTURE DIP**) |
| 25   | Docker + Nginx + TLS | 30 / 70 | AI is great at YAML; you verify every port |
| 26   | CI/CD + Projects 5/6 | 25 / 75 | AI scaffolds, you review the pipeline |
| 27   | Capstone architecture | 55 / 45 | Design is human work (**ARCHITECTURE DIP**) |
| 28   | Capstone multi-tenant core | 40 / 60 | AI cannot keep tenants safe |
| 29   | Capstone integrations | 25 / 75 | AI is your hands; you are the brain |
| 30   | Capstone launch + demo | 20 / 80 | Engineer-led, AI-powered |

Two deliberate dips break the otherwise steady loosening:

- **Week 18** bumps manual back up for webhook security and idempotency. The "never trust AI with security" rule applies hard.
- **Week 24 and Week 27** bump manual back up for architecture work. "Architecture is human work" -- AI cannot taste system design.

---

## The Permanent Rules

Five rules activate during the programme and stay active forever. Every later week respects them.

1. **Never trust AI with money** (introduced Week 10). Every line that touches an amount, a reference, or an idempotency key is hand-written and hand-reviewed.
2. **Never trust AI with auth** (introduced Week 12). Every line that hashes, signs, verifies, or enforces a permission is hand-written and hand-reviewed.
3. **Never trust AI with tenant isolation** (introduced Week 28). Every line that sets or checks a tenant boundary is hand-written and hand-reviewed.
4. **Architecture is human work** (reinforced Weeks 24 and 27). System design decisions come from you, always.
5. **If you wrote it, you must be able to explain it.** If AI typed it and you cannot explain it, you did not write it.

---

## The Weekly Habits, Stacked

By Week 30 a student has internalised twelve distinct AI habits. They stack, they do not replace each other. The full list:

1. **Week 1 -- Ask AI to explain, not to do.** Every prompt ends in a question mark.
2. **Week 2 -- Compare and contrast.** Your version first, AI version second, judge both.
3. **Week 3 -- The 15-minute rule.** Try for 15 minutes before prompting.
4. **Week 4 -- Build first, enhance with AI.** Manual version must exist and work before AI touches it.
5. **Week 5 -- AI as code reviewer.** Paste your working code, ask for critique, decide what to accept.
6. **Week 6 -- Read the docs before the prompt.** Open MDN first, every time.
7. **Week 7 -- New framework, manual first.** First 3-5 components by hand before AI scaffolds anything.
8. **Week 8 -- AI as pair partner.** Ask follow-ups, push back, decide yourself.
9. **Week 9 -- AI reads docs, you make decisions.** Architecture choices stay yours.
10. **Week 10 -- Never trust AI with money.** Permanent rule.
11. **Week 12 -- Never trust AI with auth.** Permanent rule.
12. **Week 30 -- Engineer-led, AI-powered.** The target state. You drive; AI accelerates.

Several weeks introduce additional, narrower habits (protocol-first for integrations, paper-first for state machines, handshake-first for new channels). All of them reinforce the twelve above.

---

## The AI Audit System

Every week from Week 2 onwards, students submit an `AI_AUDIT.md` file with every project. The audit is the artefact that proves a student is using AI with judgement.

### Standard audit format

```markdown
# AI Audit -- Week [X] Project

## AI tools used
- [Tool name and version]

## Summary
- Total files in project: [X]
- Files with AI-generated code: [X]
- Estimated % of code AI-generated: [X%]

## AI usage log

### [Feature / Component name]
- **Prompt used:** [exact prompt or summary]
- **What AI generated:** [brief description]
- **What I changed and why:** [specific changes]
- **What I learned:** [1-2 sentences]
- **Would I prompt differently next time?** [yes/no and why]
```

### Week-specific additions

Most weeks add one or more required sections:

- **Week 2:** Comparisons log -- mine vs AI, which I picked and why
- **Week 3:** What I tried in the 15 minutes before asking
- **Week 4:** Build-first log -- pseudocode, manual first version, then AI extension
- **Week 5:** Code review decisions -- what I accepted from AI review, what I rejected
- **Week 6:** MDN link read first before every prompt
- **Week 7:** Pattern scaffolding log
- **Week 8:** Pair conversation log with pushback
- **Week 9:** Decisions I owned
- **Week 10:** Money code audit -- who wrote every line that touches money
- **Week 11:** Split log -- manual vs AI per feature, classified
- **Week 12:** Auth manual-only proof
- **Week 13:** Protocol quirks log
- **Week 14:** Server/Client classification log (Next.js)
- **Week 15:** Photo of paper state-machine diagram
- **Week 16:** Wiring diagram
- **Week 17:** Provider quirks log
- **Week 18:** Security audit per file, line-by-line
- **Week 19:** Final public API surface with design ownership
- **Week 20:** Handshake log
- **Week 21:** Cron reliability table
- **Week 22:** Paper-drawn domain model
- **Week 23:** Job type table with priority, retry, rate limit, designer
- **Week 24:** Paper diagram of service split
- **Week 25:** Line-by-line config review
- **Week 26:** Workflow trigger + secrets + env audit
- **Week 27:** Paper diagram + API doc + schema + MILESTONES.md
- **Week 28:** RLS coverage matrix
- **Week 29:** Cross-tenant test checklist
- **Week 30:** Self-assessment: am I an engineer-led, AI-powered developer?

**Audit grading weight:** 20% of the weekly project grade. A project with excellent code but a lazy audit loses 20%.

---

## The "Break-It" Challenges

Every two weeks (Weeks 2, 4, 6, 8, 10, 12, and continuing through 14, 16, 18, 22, 26, 30), students receive a **Break-It Challenge**: a deliberately broken codebase with realistic, subtle bugs that students must debug without AI.

The challenges scale with the student's progression:

- **Week 2:** CSS file with specificity and cascade bugs
- **Week 4:** JavaScript function with off-by-one and scope bugs
- **Week 6:** Async code with stale closures and race conditions
- **Week 8:** React form with state bugs and missing dependencies
- **Week 10:** Express API with missing amount validation
- **Week 12:** Auth system with JWT never expiring, plaintext password in one route, CORS wide open
- **Week 14:** Next.js app with wrong Server/Client classification
- **Week 16:** Payment integration with a race condition in the webhook handler
- **Week 18:** Webhook handler with signature bypass and missing idempotency
- **Week 22:** Chama bot with broken fine computation and cycle transition
- **Week 26:** CI/CD pipeline deploying from a feature branch by accident
- **Week 30:** Full capstone-shaped app with 10 bugs across frontend, backend, database, security, and tenant isolation

**Rules:** No AI during Break-It Challenges. 60 minutes. Fix what you can, document what you found but could not fix.

---

## Assessment Framework

Each week, students are evaluated on four dimensions:

1. **Understanding (30%)** -- Can you explain what the code does and why? Tested via verbal/written explanation, whiteboard sessions, or Break-It challenges.
2. **Implementation (30%)** -- Does the project work? Is it complete? Is it production-quality? Code review by facilitator.
3. **AI Literacy (20%)** -- Quality of AI Audit. Did you use AI effectively? Did you catch AI mistakes? Did you learn from the process?
4. **Engineering Judgment (20%)** -- Architecture decisions, security awareness, error handling, code organisation. Did you make good decisions, regardless of who (or what) wrote the code?

Every week also includes a **Live Rebuild Check** during assessment: the facilitator deletes one piece of the student's code and asks them to rewrite it from memory. If they cannot, that is not automatic failure -- it is a flag to revisit next week. Repeated flags trigger a one-on-one review.

---

## The Phase Structure

The 30 weeks group into five phases. Each phase closes with a major project.

### Phase 1: Setup and Web Fundamentals (Weeks 1-6)
**Ratio range:** 90/10 -> 65/35
**Focus:** tools, HTML, CSS, JavaScript basics, async, DOM.
**Habit stack built:** explain-not-do, compare-and-contrast, 15-minute rule, build-first, AI-as-reviewer, docs-first.
**Deliverable:** Weekend projects, Data Dashboard (Week 6).

### Phase 2: React + Backend + First Integrations (Weeks 7-13)
**Ratio range:** 60/40 -> 45/55
**Focus:** React, forms, routing, state, custom hooks, testing, M-Pesa, WhatsApp, Postgres, auth, USSD.
**New permanent rules:** Never trust AI with money (Week 10), never trust AI with auth (Week 12).
**Deliverables:** Project 1 (M-Pesa Paylink, Week 10), Project 2A (WhatsApp CRM, Week 11), Project 2B (USSD support, Week 13).

### Phase 3: Full-Stack Shop + Commerce (Weeks 14-19)
**Ratio range:** 40/60 -> 35/65
**Focus:** Next.js, cart, checkout state machine, multi-provider payments, webhook security, reusable payments library.
**Security dip:** Week 18.
**Deliverable:** Project 3 (E-commerce + WhatsApp, Week 16), published payments package (Week 19).

### Phase 4: Automation + Infrastructure (Weeks 20-26)
**Ratio range:** 40/60 -> 25/75
**Focus:** Telegram bots, cron, queues, microservices, Docker, CI/CD.
**Architecture dip:** Week 24.
**Deliverables:** Project 4 (Chama Savings, Week 22), Projects 5 (Booking) and 6 (Notification Hub) in Week 26.

### Phase 5: Capstone Sprint (Weeks 27-30)
**Ratio range:** 55/45 -> 20/80
**Focus:** Multi-tenant SaaS (African SME OS or student-chosen).
**Architecture dip:** Week 27.
**New permanent rule:** Never trust AI with tenant isolation (Week 28).
**Deliverable:** Live production SME OS with USSD, WhatsApp, payments, wallets, dashboard.

---

## Notes For Facilitators

- **Never ban AI outright.** Banning creates a black market. Structure its use instead.
- **Spot-check AI Audits.** Read the prompts students used. You will learn more about their understanding from their prompts than from their code.
- **Celebrate good AI usage.** When a student uses AI brilliantly -- great prompt, caught an AI mistake, used AI to learn faster -- highlight it publicly. This normalises productive AI usage.
- **Watch for the "AI plateau".** Some students will hit a ceiling where they can build anything with AI but cannot debug or extend without it. This usually shows up around Weeks 7-8. The Break-It challenges are designed to catch this.
- **The goal is judgement, not speed.** A student who takes longer but understands deeply will outperform a student who ships fast but cannot maintain what they built. Assess accordingly.
- **Enforce the permanent rules ruthlessly.** A student who lets AI write a JWT verification cannot be allowed to pass the week. Send them back to do it by hand. It is a service to them.
- **Reward paper work.** Paper diagrams, whiteboard photos, hand-written state machines -- these are signs of engineering judgement. Value them in grading.

---

## Notes For Students

- **Read the per-week `ai.md` before Day 1 of every week.** It tells you what is manual, what is AI-allowed, and what this week's habit is.
- **Keep your AI Audit honest.** Lying in the audit is worse than being slow. Facilitators can always spot the gap between your code and your claimed understanding.
- **Every habit is permanent.** Week 1's "ask AI to explain" still applies in Week 30.
- **The permanent rules save your job.** Money, auth, tenant isolation, architecture -- these are the four things that, when done wrong, end careers. Do them by hand.
- **Ship things.** The best evidence that you learned is a working product with real users. Do the weekend projects. Do the capstone. Ship.

---

## One Sentence For The Whole Programme

"Engineer-led, AI-powered." You drive. AI accelerates. Anything else is either ego or dependence.
