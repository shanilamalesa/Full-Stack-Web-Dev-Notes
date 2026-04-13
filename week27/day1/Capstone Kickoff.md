# Week 27, Day 1: Capstone Kickoff

> **AI boundaries this week:** 55% manual / 45% AI (ARCHITECTURE DIP). Habit: *Design is human work -- pen beats keyboard.* See [ai.md](../ai.md).

Welcome to the Capstone Sprint. Four weeks. One real project. No more daily lessons of new concepts -- this phase is about applying everything you have learned to a single ambitious build.

**Estimated time:** 2 hours (reading + planning).

---

## The Capstone Sprint Shape

| Week | Focus |
|---|---|
| 27 | Architecture, schema, API design, repo setup |
| 28 | Core multi-tenant platform and auth |
| 29 | Integrations: USSD, WhatsApp, payments |
| 30 | Polish, testing, documentation, demo day |

No more "Day 1 teaches X, Day 2 teaches Y". Each day is a milestone, not a lesson. The notes for the Capstone are shorter; you are in driving seat.

---

## The Default Idea: The African SME OS

A multi-tenant platform where a business owner signs up and gets:

- A USSD code they can put on a poster (`*384*1234#`).
- A WhatsApp bot customers can message.
- A web dashboard to see orders, messages, and balance.
- A Wallet with M-Pesa in and out.

One signup. Three channels. One unified database. The "SME OS" is the thing a small Kenyan business can plug into without stitching five tools together themselves.

### Why this target

- It uses everything from Phases 1-4 (Postgres, Next.js, USSD, WhatsApp, payments, queues, Docker).
- It is multi-tenant from day one, which forces you to think about isolation.
- It is a real product idea -- if you ship it well, it is launchable.
- It is ambitious enough to be a CV anchor piece.

---

## Bring Your Own Idea (If Aligned)

You can swap the SME OS for your own project, with constraints:

- Must be multi-tenant (multiple customers each getting their own space).
- Must integrate at least two of: USSD, WhatsApp, M-Pesa, Telegram.
- Must have a Next.js dashboard.
- Must deploy end-to-end by Week 30 Day 5.

If your idea does not hit those, do the SME OS instead. This is not about creativity, it is about execution on a complete system.

---

## Week 27 Milestones

By Friday night of this week, you should have:

1. **An ARCHITECTURE.md** in the repo root with the system diagram, service list, and decisions.
2. **A SCHEMA.sql** with every table for the core system (no integration tables yet).
3. **An API design document** listing every route with method, path, body, response shape.
4. **A fresh git repo** with a scaffolding commit and placeholder folders for each service.
5. **A backlog** of tasks for Weeks 28-30, roughly estimated.

This week writes very little production code. The goal is to reach Monday of Week 28 with a clear path to follow.

---

## Core Questions To Answer Today

Write the answers in your notebook or in `DESIGN.md`. These are the questions every multi-tenant platform eventually answers; doing it now saves pain later.

1. **Who is a tenant?** A company? A person? A "space"? Pick one.
2. **How do tenants sign up?** Free self-serve? Invite only? Pay first?
3. **What is the isolation boundary?** Row-level filters? Schema per tenant? Database per tenant?
4. **What happens to data if a tenant deletes their account?** Soft delete? Hard delete after 30 days? Exported?
5. **Can one user belong to multiple tenants?** (SaaS patterns: usually yes.)
6. **What is the minimum feature set a tenant must have on their first day?** (Keep this small.)
7. **What is the upgrade path to paid?** (For the SME OS, what makes it compelling enough to pay?)
8. **What is the data volume you expect?** 100 tenants? 10,000? Different numbers demand different architectures.

Write the answers out. Rewrite them tomorrow if you change your mind. Keep the doc under a page -- this is a north star, not a spec.

---

## The Stack You Will Use

Based on everything built so far:

- **Postgres** for everything relational.
- **Redis** for sessions, queues, caches.
- **Next.js 14** for the dashboard.
- **Express** for the backend API, webhooks, and USSD endpoint.
- **Separate notification-service** for outbound messaging.
- **BullMQ** for all async work.
- **Docker Compose** for local dev and single-server production.
- **GitHub Actions** for CI/CD.
- **`@mctaba/payments`** package from Week 19 for money handling.

You are not learning any new tech this sprint. You are applying everything you already know to a bigger codebase than anything so far.

---

## Setting Up The Repo

```bash
mkdir sme-os
cd sme-os
git init
```

Create the folders:

```bash
mkdir -p api notification-service shop contracts nginx docs scripts sql
touch README.md ARCHITECTURE.md docker-compose.yml .gitignore
```

`README.md` starts with one paragraph describing the project. `ARCHITECTURE.md` will grow through the week.

Commit the scaffolding:

```bash
git add .
git commit -m "init: sme-os repo scaffolding"
```

---

## Backlog Preview

Rough task list for the four weeks. Do not worry about ordering or estimates today.

- Tenant signup.
- Tenant schema and isolation.
- Auth (admin and staff users per tenant).
- Storefront / portal that each tenant can link to their customers.
- USSD code routing by tenant.
- WhatsApp bot per tenant (one bot, many tenants).
- M-Pesa till/paybill per tenant.
- Wallet: in (from customers), out (to tenants).
- Dashboard: orders, messages, wallet balance, settings.
- Deployment.
- Demo rehearsal.

Estimate 20-30 discrete tasks. Tomorrow we turn these into a milestone plan for each of the four weeks.

---

## Checkpoint

1. You read this doc and know the shape of the Capstone.
2. You picked the SME OS or your own aligned idea.
3. You have a fresh `sme-os/` repo with empty folders.
4. You have a `DESIGN.md` with answers to the eight questions.
5. You have a rough backlog of 20-30 tasks.

Commit:

```bash
git add .
git commit -m "docs: kickoff design and backlog"
```

---

## What You Learned

Nothing new. Today is about clearing the mental space to build something real. Tomorrow we design the data model for multi-tenancy, which is the single most important decision of the Capstone.
