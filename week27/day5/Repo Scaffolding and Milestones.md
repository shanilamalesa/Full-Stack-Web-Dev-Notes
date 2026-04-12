# Week 27, Day 5: Repo Scaffolding and Milestones

By the end of today, the SME OS repo is fully scaffolded -- every service has a `package.json`, every Dockerfile, `docker-compose.yml`, `sql/schema.sql` loads, and a clear milestone plan for Weeks 28-30 lives in `docs/PLAN.md`.

**Estimated time:** 3 hours.

---

## The Folder Tree

```
sme-os/
  api/
    package.json
    index.js
    config/
    controllers/
    services/
    repositories/
    routes/
    middleware/
    Dockerfile
    .env.example
  notification-service/
    package.json
    index.js
    workers/
    senders/
    Dockerfile
  shop/                      # the tenant dashboard, next.js
    package.json
    app/
    lib/
    next.config.js
    Dockerfile
  contracts/
    package.json
    index.js
  sql/
    schema.sql
    seed.sql
  nginx/
    nginx.conf
    sites/smeos.conf
  docker-compose.yml
  docker-compose.override.yml
  docs/
    ARCHITECTURE.md
    API.md
    PLAN.md
  scripts/
    reset-db.sh
  .github/workflows/
    ci.yml
    publish.yml
  .gitignore
  README.md
```

Everything from Weeks 12-26 applied to one fresh repo. Copy as much as you can from earlier projects -- do not reinvent.

---

## What To Copy From Earlier Weeks

- **Week 12 CRM scaffolding** -- routes/controllers/services/repositories layout.
- **Week 14-16 shop** -- Next.js App Router setup, Tailwind config, auth cookies.
- **Week 18 payment hardening** -- webhook verification, outbox, idempotency.
- **Week 19 payments package** -- install as a dependency from your published version.
- **Week 20 Telegram bot** -- if you need Telegram; otherwise skip.
- **Week 23 BullMQ queues** -- directly.
- **Week 24 split** -- keep notification-service separate.
- **Week 25 Docker** -- copy the Dockerfiles and compose file.
- **Week 26 CI/CD** -- copy the GitHub Actions workflows.

Aim for the first commit of real work to look like a finished Week 25 project, just with a different domain model.

---

## The Milestone Plan

```markdown
# Capstone Milestones

## Week 28 (Core platform and auth)

### Day 1
- Tenant signup endpoint.
- User signup and login.
- Tenant creation creates a wallet row and a default set of products.

### Day 2
- Tenant membership + roles.
- Auth middleware that loads tenant and sets RLS context.
- First protected endpoint: GET /api/customers.

### Day 3
- Products CRUD.
- Orders CRUD (without payment yet).
- Order state transitions.

### Day 4
- Wallet read endpoints.
- Wallet transaction logging with balance_after_cents.
- Smoke test: two tenants fully isolated end-to-end.

### Day 5
- Dashboard shell in Next.js with tenant switcher.
- Login flow.
- Empty orders table view.
- Week 28 demo ready.

### Weekend
- Polish the week's features.
- Deploy to a staging server.
- Take screenshots.

## Week 29 (Integrations)

### Day 1
- USSD webhook routes by tenant (based on short code).
- First USSD menu that reads from the tenant's products.

### Day 2
- WhatsApp webhook routes by tenant (based on phone number id).
- Inbound message captured as a lead.

### Day 3
- M-Pesa STK push per tenant (each tenant has its own shortcode or bills through yours).
- Order payment flow.

### Day 4
- Dashboard integration: orders from USSD and WhatsApp appear in one inbox.
- Wallet updates on payment confirmation.

### Day 5
- Full demo: customer dials USSD, buys a product, gets a WhatsApp confirmation, the tenant sees the order in the dashboard.
- Week 29 demo ready.

### Weekend
- Fix bugs from the Week 29 demo.
- Deploy to staging.

## Week 30 (Polish and launch)

### Day 1
- Testing: unit + integration tests for the critical paths.
- Load test with 10 fake tenants and 100 orders each.

### Day 2
- Documentation: user-facing docs, ops runbook, API docs.
- Landing page for the SME OS.

### Day 3
- Final deploy.
- DNS pointing to production.
- Let's Encrypt certs.

### Day 4
- Demo rehearsal.
- Bug-fix window.
- Final content polish.

### Day 5
- Demo day.
- Ship it.

### Weekend
- Reflection, next steps, decompression.
```

Write this in `docs/PLAN.md`. Adjust as needed -- nothing here is set in stone. The value is having one clear plan everyone agrees on.

---

## A Sample First Commit

The goal by end of day:

```bash
git log --oneline
a1b2c3d  docs: api spec with thirty endpoints
4d5e6f7  docs: schema with rls policies
7g8h9i0  docs: multi-tenancy pattern and rls plan
j1k2l3m  docs: kickoff design and backlog
n4o5p6q  init: sme-os repo scaffolding
```

Five commits. Every day of Week 27 added one. Small commits, clear messages.

---

## Checkpoint

1. `sme-os/` has the full folder structure.
2. `docker-compose up -d` starts Postgres and Redis (no app services yet).
3. `sql/schema.sql` loads.
4. Each service folder has a `package.json` and `Dockerfile`.
5. `docs/PLAN.md` has a day-by-day milestone list for Weeks 28-30.
6. The GitHub repo is up and a branch is pushed.

Commit:

```bash
git add .
git commit -m "chore: scaffold all services and milestone plan"
git push
```

---

## Week 27 Recap

A week of mostly thinking and spec'ing. Nothing ships. But by Friday night you know exactly what to build for the next three weeks.

### What you produced
- `DESIGN.md` with answers to the eight core questions.
- `ARCHITECTURE.md` with the multi-tenancy pattern.
- `sql/schema.sql` with ten tables and RLS.
- `docs/API.md` with thirty endpoints.
- `docs/PLAN.md` with a day-by-day milestone list.
- Full repo scaffold.
- CI workflow copied from Week 26.

### What is next
- Monday of Week 28: start shipping code against the plan.
- No more spec changes this week -- commit to what you wrote.

Weekend project: `week27/weekend/project.md`.
