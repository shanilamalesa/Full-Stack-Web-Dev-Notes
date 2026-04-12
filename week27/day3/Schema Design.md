# Week 27, Day 3: Schema Design

By the end of today, `sql/schema.sql` has every core table for the SME OS, every table has RLS enabled, and the schema loads cleanly on a fresh Postgres database.

**Estimated time:** 3 hours.

---

## The Tables

Start with what every SME OS tenant needs:

1. `tenants` -- the companies.
2. `users` -- people (can belong to multiple tenants).
3. `tenant_members` -- linking users to tenants with roles.
4. `customers` -- end customers of each tenant's business.
5. `orders` -- customer orders.
6. `order_items` -- line items per order.
7. `products` -- the tenant's catalogue.
8. `messages` -- unified inbox across channels.
9. `wallets` -- each tenant's account balance.
10. `wallet_transactions` -- every in/out movement.

Ten tables. Add more as the integrations land in Week 29.

---

## The Full Schema

```sql
-- Extensions
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS btree_gist;

-- Tenants
CREATE TABLE tenants (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  slug TEXT NOT NULL UNIQUE,
  ussd_code TEXT UNIQUE,
  whatsapp_phone_number_id TEXT,
  mpesa_shortcode TEXT,
  status TEXT NOT NULL DEFAULT 'active'
    CHECK (status IN ('active', 'suspended', 'deleted')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ
);

-- Users (global, not per tenant)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  password_hash TEXT NOT NULL,
  name TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tenant membership
CREATE TABLE tenant_members (
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role TEXT NOT NULL CHECK (role IN ('owner', 'admin', 'staff')),
  joined_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  PRIMARY KEY (tenant_id, user_id)
);

-- Customers (per tenant)
CREATE TABLE customers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  phone TEXT NOT NULL,
  name TEXT,
  first_seen_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (tenant_id, phone)
);

CREATE INDEX idx_customers_tenant ON customers(tenant_id);

-- Products
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  sku TEXT NOT NULL,
  name TEXT NOT NULL,
  price_cents INTEGER NOT NULL CHECK (price_cents >= 0),
  stock INTEGER NOT NULL DEFAULT 0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (tenant_id, sku)
);

CREATE INDEX idx_products_tenant ON products(tenant_id);

-- Orders
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  customer_id UUID REFERENCES customers(id),
  total_cents INTEGER NOT NULL,
  channel TEXT NOT NULL CHECK (channel IN ('ussd', 'whatsapp', 'web')),
  status TEXT NOT NULL DEFAULT 'pending'
    CHECK (status IN ('pending', 'paid', 'delivered', 'cancelled')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_orders_tenant_created ON orders(tenant_id, created_at DESC);
CREATE INDEX idx_orders_tenant_status ON orders(tenant_id, status);

-- Order items
CREATE TABLE order_items (
  id BIGSERIAL PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id UUID REFERENCES products(id),
  quantity INTEGER NOT NULL,
  price_cents_at_purchase INTEGER NOT NULL
);

CREATE INDEX idx_order_items_order ON order_items(order_id);

-- Messages (unified inbox)
CREATE TABLE messages (
  id BIGSERIAL PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  customer_id UUID REFERENCES customers(id),
  channel TEXT NOT NULL,
  direction TEXT NOT NULL CHECK (direction IN ('in', 'out')),
  body TEXT,
  raw_payload JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_messages_tenant_created ON messages(tenant_id, created_at DESC);
CREATE INDEX idx_messages_customer ON messages(customer_id);

-- Wallets
CREATE TABLE wallets (
  tenant_id UUID PRIMARY KEY REFERENCES tenants(id) ON DELETE CASCADE,
  balance_cents BIGINT NOT NULL DEFAULT 0 CHECK (balance_cents >= 0),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Wallet transactions
CREATE TABLE wallet_transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  amount_cents BIGINT NOT NULL,     -- positive for in, negative for out
  kind TEXT NOT NULL CHECK (kind IN ('credit', 'debit', 'fee', 'adjustment')),
  ref TEXT,                         -- external reference (M-Pesa receipt, etc)
  note TEXT,
  balance_after_cents BIGINT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_wallet_txn_tenant ON wallet_transactions(tenant_id, created_at DESC);

-- Enable RLS on all tenant-scoped tables
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE products ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE order_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE wallets ENABLE ROW LEVEL SECURITY;
ALTER TABLE wallet_transactions ENABLE ROW LEVEL SECURITY;

-- Policies
CREATE POLICY tenant_isolation ON customers USING (tenant_id = current_setting('app.current_tenant')::uuid);
CREATE POLICY tenant_isolation ON products USING (tenant_id = current_setting('app.current_tenant')::uuid);
CREATE POLICY tenant_isolation ON orders USING (tenant_id = current_setting('app.current_tenant')::uuid);
CREATE POLICY tenant_isolation ON order_items USING (tenant_id = current_setting('app.current_tenant')::uuid);
CREATE POLICY tenant_isolation ON messages USING (tenant_id = current_setting('app.current_tenant')::uuid);
CREATE POLICY tenant_isolation ON wallets USING (tenant_id = current_setting('app.current_tenant')::uuid);
CREATE POLICY tenant_isolation ON wallet_transactions USING (tenant_id = current_setting('app.current_tenant')::uuid);
```

A lot of lines. Each one deliberate.

---

## A Few Design Notes

**`tenant_id` on `order_items`.** Technically redundant because you could join through `orders`. But it simplifies RLS -- every table filters identically. The cost is a small amount of storage. The benefit is clarity and speed. Worth it.

**`wallets.balance_cents` as `BIGINT`.** A successful SME OS tenant might have 100,000,000 cents (KSh 1M) in their wallet. A very successful one might have 10 billion cents. `INTEGER` caps at 2.1B -- too small. `BIGINT` caps at 9.2 quintillion. Use it for money at scale.

**`wallet_transactions.balance_after_cents` stored.** Derivable from sum of prior transactions, but storing it lets you show a running balance on a statement page without a recursive sum. This is a deliberate denormalisation for performance.

**Soft delete tenants with `deleted_at`.** A deleted tenant is still in the table for 30 days. RLS policies should add `AND tenants.deleted_at IS NULL` for the user-facing paths but let admin see everything.

---

## Loading The Schema

```bash
cd sme-os
docker run --rm -v $(pwd)/sql:/sql postgres:16 psql -h host.docker.internal -U crm_user -d crm -f /sql/schema.sql
```

Or, simpler, if you have psql installed locally:

```bash
psql -U crm_user -d smeos -f sql/schema.sql
```

Verify:

```sql
\dt
-- lists tables

\d tenants
\d orders
-- shows the details

SELECT * FROM pg_policies;
-- lists RLS policies
```

All seven policies should appear.

---

## Smoke Test RLS

Create two tenants and verify isolation:

```sql
INSERT INTO tenants (name, slug) VALUES ('Tenant A', 'a'), ('Tenant B', 'b');

INSERT INTO customers (tenant_id, phone, name) VALUES
  ((SELECT id FROM tenants WHERE slug = 'a'), '+254711000001', 'Customer A1'),
  ((SELECT id FROM tenants WHERE slug = 'b'), '+254711000002', 'Customer B1');

-- As tenant A
SET app.current_tenant = (SELECT id::text FROM tenants WHERE slug = 'a');
SELECT * FROM customers;  -- should only show A1

-- As tenant B
SET app.current_tenant = (SELECT id::text FROM tenants WHERE slug = 'b');
SELECT * FROM customers;  -- should only show B1

-- Without the setting
RESET app.current_tenant;
SELECT * FROM customers;
-- error or empty, depending on your pg version
```

The last line is the critical test: without setting the tenant, the query should not return anything (or should error). Postgres RLS with a `USING` policy rejects rows that do not match -- and `current_setting('app.current_tenant')` throws if the variable is not set. The result: a request that forgets to set the tenant cannot see any data.

This is the safety net. Revisit it often.

---

## Checkpoint

1. `sql/schema.sql` loads on a fresh database without errors.
2. Ten tables exist.
3. Seven RLS policies are visible via `SELECT * FROM pg_policies`.
4. The smoke test above shows strict tenant isolation.
5. `ARCHITECTURE.md` links to the schema file.

Commit:

```bash
git add sql/schema.sql
git commit -m "feat: core schema with rls policies"
```

---

## What You Learned

- Tenant-scoped tables all carry `tenant_id`.
- RLS policies are one line per table.
- Denormalising `balance_after_cents` is worth the clarity.
- Soft deletes keep options open.
- The smoke test proves RLS works end to end.

Tomorrow: API design.
