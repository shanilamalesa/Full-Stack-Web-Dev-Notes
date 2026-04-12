# Week 27, Day 2: Multi-tenant Patterns

By the end of today, you have picked the multi-tenancy strategy for the SME OS, written it down, and understood the trade-offs of each pattern.

**Estimated time:** 2 hours.

---

## The Three Patterns

There are three commonly used strategies for multi-tenant data isolation. Each has clear trade-offs.

### 1. Shared tables with a `tenant_id` column

Every table has a `tenant_id` foreign key. Every query filters by `tenant_id`. One database, one schema, rows belong to tenants by label.

**Pros:**
- Simplest to implement.
- Easy to add a new tenant (no schema changes).
- Efficient at scale (one connection pool, one cache).
- Easy to run cross-tenant analytics.

**Cons:**
- Every query must remember to filter. A single `SELECT * FROM orders` without a tenant filter is a leak.
- Hard to give a tenant a backup of "just their data".
- Noisy neighbors: one tenant's big query can slow down others.

### 2. Schema per tenant

Each tenant gets their own Postgres schema (`tenant_abc.orders`, `tenant_xyz.orders`). Same database, different namespaces.

**Pros:**
- Strong isolation: queries target a specific schema.
- Easy to backup/restore per tenant.
- Easy to migrate one tenant at a time.

**Cons:**
- Migrations are O(tenants) -- adding a column means running the same DDL N times.
- Connection pool gets complicated because you need to `SET search_path` per request.
- Does not scale beyond a few hundred tenants comfortably.

### 3. Database per tenant

Each tenant has its own Postgres database (or its own server).

**Pros:**
- Strongest isolation.
- Noisy neighbors are impossible.
- Easiest to comply with "data sovereignty" requirements.

**Cons:**
- Massive operational overhead.
- Impossible cross-tenant analytics without a separate pipeline.
- Ridiculously expensive if you have more than ~20 tenants.

---

## What We Pick For The SME OS

**Pattern 1 -- shared tables with `tenant_id`** -- with **Postgres row-level security** enforcing isolation at the database level.

Reasoning:

- We expect hundreds to thousands of SMEs, not tens. Pattern 2 does not scale to that.
- We want simple ops. Pattern 3 is too expensive for the target.
- The "every query must filter" footgun is solved by RLS: even if you forget the filter, Postgres refuses to return other tenants' rows.

This is the pattern used by Linear, Notion, GitHub, and every modern multi-tenant SaaS. It is proven at scale and with the right safeguards it is safe.

---

## Row-Level Security Basics

Postgres's RLS lets you write a policy per table that says "only rows where this condition is true are visible to this user".

```sql
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON orders
  USING (tenant_id = current_setting('app.current_tenant')::uuid);
```

Now every `SELECT`, `UPDATE`, `DELETE` on `orders` silently filters by the current tenant. Even `SELECT *` returns only the current tenant's rows.

How does Postgres know the current tenant? You set a session variable at the start of every request:

```javascript
// In a middleware:
await client.query(`SET LOCAL app.current_tenant = $1`, [tenantId]);
```

Every query after this on the same client sees only that tenant's rows. When the transaction ends, the variable clears.

---

## What RLS Protects You From

- A bug in the repo layer that forgets a `WHERE tenant_id = ...`.
- A SQL injection that tries to read other tenants' data.
- A broken JOIN that leaks across tenants.
- A cron job that forgets to scope by tenant.

What RLS does NOT protect you from:

- Setting the wrong tenant id in the first place (authentication bugs).
- Administrative users who bypass RLS intentionally.
- Indexes that leak information through timing attacks (academic but real).

Layered defense: RLS is the safety net, correct code is the ground. Both are needed.

---

## The `tenant_id` Column Everywhere

Add `tenant_id UUID NOT NULL REFERENCES tenants(id)` to every data table:

```sql
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT NOT NULL UNIQUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  customer_phone TEXT NOT NULL,
  total_cents INTEGER NOT NULL,
  status TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_orders_tenant ON orders(tenant_id);

ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON orders
  USING (tenant_id = current_setting('app.current_tenant')::uuid);
```

Three lines of RLS per table. Boring but necessary. A small helper can generate the policy for every new table in a migration.

### Super user access

Some operations (reconciliation, cross-tenant analytics) need to see everything. Create a "bypass" role:

```sql
CREATE ROLE app_admin BYPASSRLS;
```

A connection from the admin role ignores RLS. Only cron jobs and the reconciliation worker use this role; everything else uses the regular role that RLS applies to.

---

## Tenant Resolution: How Does The Request Know Its Tenant

Four common ways:

1. **Subdomain**: `tenant-a.smeos.co.ke` resolves to tenant "a". Clean URLs, requires wildcard DNS.
2. **Path prefix**: `smeos.co.ke/tenant-a/...` -- easier to set up but uglier URLs.
3. **Session-based**: user logs in, session has `tenant_id`. Best for dashboards where users are authenticated.
4. **Header-based**: `X-Tenant-Id: abc` -- for API clients.

The SME OS uses (3) for the dashboard and (1) or (2) for public pages. Pick based on your preference -- path prefix is simpler.

---

## Write It Down

Update `ARCHITECTURE.md` with:

```markdown
## Multi-tenancy

Pattern: shared tables with `tenant_id` column, row-level security enforcing isolation.

Tenant resolution:
- Dashboard: session contains `tenant_id`, middleware sets `app.current_tenant` before each query.
- Public pages: URL path prefix (`/t/tenant-slug/...`).
- API: `Authorization: Bearer <token>` contains tenant claim.

Isolation guarantee: RLS prevents cross-tenant data leaks even in the face of repo bugs.

Bypass: `app_admin` role for cron and reconciliation.

Deletion: soft-delete tenants for 30 days; hard-delete via cron.
```

Commit.

---

## Checkpoint

1. You picked shared tables + RLS for the Capstone.
2. You understand how `current_setting` works for tenant resolution.
3. You know which operations need the bypass role and which do not.
4. `ARCHITECTURE.md` has a multi-tenancy section.

Commit:

```bash
git add ARCHITECTURE.md
git commit -m "docs: multi-tenancy pattern and rls plan"
```

---

## What You Learned

- Three multi-tenancy patterns, one sensible default.
- Row-level security is the Postgres feature that makes shared tables safe.
- `current_setting` is the Postgres equivalent of "current user" scoping.
- Tenant resolution comes from URL, session, or header.
- Write the decision down before writing code.

Tomorrow: schema design.
