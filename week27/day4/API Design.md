# Week 27, Day 4: API Design

By the end of today, `docs/API.md` lists every endpoint the SME OS exposes, with path, method, body, and response shape. You will not write any handlers today -- just the spec.

**Estimated time:** 2-3 hours.

---

## Why Spec First

Writing handlers before you know what you are writing leads to inconsistent APIs, duplicate endpoints, and painful refactors. Spending two hours on a spec saves days of rework.

The spec is a living document. It will change. That is fine. The initial version is a north star for Week 28.

---

## Structure

Organise by resource. Group tenant, auth, core resources, and integrations.

```
Auth
  POST /api/auth/signup
  POST /api/auth/login
  POST /api/auth/logout
  GET  /api/auth/me

Tenant
  POST /api/tenants              (create -- signup flow)
  GET  /api/tenants/:slug        (public)
  PATCH /api/tenants/:slug       (owner only)
  POST /api/tenants/:slug/members  (invite)

Customers (tenant scoped)
  GET  /api/customers
  GET  /api/customers/:id
  POST /api/customers

Products
  GET  /api/products
  POST /api/products
  PATCH /api/products/:id
  DELETE /api/products/:id

Orders
  GET  /api/orders
  GET  /api/orders/:id
  POST /api/orders
  PATCH /api/orders/:id/status

Messages
  GET  /api/messages
  POST /api/messages

Wallet
  GET  /api/wallet
  GET  /api/wallet/transactions
  POST /api/wallet/topup        (for manual admin topups)

Integrations (webhooks)
  POST /webhook/ussd            (Africa's Talking)
  POST /webhook/whatsapp        (Meta)
  POST /webhook/mpesa/callback  (Daraja)
  POST /webhook/stripe          (Stripe)
```

Thirty endpoints. Looks like a lot; most are trivial CRUD.

---

## Conventions

Pick conventions and stick to them:

- **Base path**: `/api/...` for JSON APIs, `/webhook/...` for webhooks, `/` for HTML pages (Next.js).
- **Methods**: `GET` for reads, `POST` for create, `PATCH` for partial update, `DELETE` for delete.
- **Response shape**: always `{ data: ..., error: null }` on success, `{ data: null, error: { message, code } }` on failure.
- **Pagination**: `?limit=20&offset=0` with response including `{ total, limit, offset }`.
- **Filtering**: query string (`?status=paid&from=2026-04-01`).
- **Auth**: JWT in `Authorization: Bearer ...` header.
- **Tenant**: derived from the authenticated user's session, not passed in the URL.

Document these in the top of `API.md`. Reviewers should be able to predict the shape of an endpoint without looking at the spec.

---

## A Full Endpoint Example

```markdown
### POST /api/orders

Create a new order.

**Auth**: required. User must be a member of the tenant.

**Request body**:
```json
{
  "customerPhone": "+254712345678",
  "items": [
    { "productId": "uuid", "quantity": 2 },
    { "productId": "uuid", "quantity": 1 }
  ],
  "channel": "web"
}
```

**Response (201)**:
```json
{
  "data": {
    "id": "uuid",
    "tenantId": "uuid",
    "customerId": "uuid",
    "totalCents": 2499,
    "status": "pending",
    "channel": "web",
    "createdAt": "2026-04-15T10:30:00Z",
    "items": [...]
  },
  "error": null
}
```

**Errors**:
- `400` -- invalid input (amount, phone format, etc.).
- `401` -- not authenticated.
- `403` -- not a member of this tenant.
- `404` -- product not found.
- `409` -- insufficient stock.
```

Do this for every endpoint. Yes, all thirty. Yes, it is tedious. Yes, it is worth it.

---

## Consistency Checks

After writing the spec, check:

- Every endpoint returns the same response shape.
- Every error maps to a sensible HTTP status code.
- Every mutation is idempotent or explicitly documented as not.
- Every list endpoint supports pagination.
- Every filterable field is documented.
- Every endpoint says which auth it requires.

If you find inconsistencies, fix them before writing code.

---

## Contracts Package

Export the shapes from your `@mctaba/contracts` package so both the API and the frontend can import them:

```javascript
// contracts/apis.js
const Order = {
  fields: {
    id: "string",
    tenantId: "string",
    customerId: "string?",
    totalCents: "number",
    status: "string",
    channel: "string",
    createdAt: "string",
    items: "array",
  },
};

const CreateOrderRequest = {
  fields: {
    customerPhone: "string",
    items: "array",
    channel: "string",
  },
};

module.exports = { Order, CreateOrderRequest };
```

The handler validates requests against `CreateOrderRequest` and types responses as `Order`. Not strict typing without TypeScript, but better than free-form.

---

## Mock Data For The Frontend

While the API is being built, the frontend can use mock data based on the spec. Create `docs/api-examples.json`:

```json
{
  "orders": [
    {
      "id": "uuid",
      "tenantId": "uuid",
      "totalCents": 2499,
      "status": "paid",
      "channel": "whatsapp",
      "createdAt": "2026-04-15T10:00:00Z"
    }
  ]
}
```

The frontend loads these in dev mode so it can be built in parallel with the API, not waiting for endpoints to exist.

---

## Checkpoint

1. `docs/API.md` has all 30 endpoints with method, path, body, response, errors.
2. Conventions are stated at the top of the doc.
3. Contracts package has types for the core resources.
4. A mock data file exists for frontend parallel work.
5. You could hand the spec to a new developer and they would know what to build.

Commit:

```bash
git add .
git commit -m "docs: api spec with thirty endpoints"
```

---

## What You Learned

- API specs prevent inconsistency.
- Conventions are more important than individual endpoints.
- Contracts package keeps API and frontend aligned.
- Mock data lets the frontend move in parallel.

Tomorrow: repo setup, initial scaffolding, and a Week 28 milestone plan.
