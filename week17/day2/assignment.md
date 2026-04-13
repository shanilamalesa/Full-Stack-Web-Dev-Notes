# Week 17 - Day 2 Assignment

## Title
Airtel Money Integration

## Overview
Today you add Airtel Money as a third payment provider. Airtel's API is less polished than Stripe's and less popular than Daraja, but it matters for some Kenyan customer segments. You will read Airtel Money developer docs first, implement the initiate flow, and handle the callback.

## Learning Objectives Assessed
- Read Airtel Money sandbox docs (less friendly than Stripe)
- Obtain an auth token from Airtel
- Initiate a collection (their name for STK push)
- Handle the callback and reconcile against your order

## Prerequisites
- Day 1 completed

## AI Usage Rules

**Ratio:** 30/70. **Habit:** Provider docs first. See [../ai.md](../ai.md).

- **ALLOWED FOR:** Scaffolding after you read the docs.
- **NOT ALLOWED FOR:** Copying endpoint URLs or body shapes without reading.
- **AUDIT REQUIRED:** Yes.

## Tasks

### Task 1: Airtel sandbox account

**What to do:**
1. Go to https://developers.airtel.africa and sign up.
2. Get your Client ID and Client Secret for the sandbox.
3. Add to `.env.local`: `AIRTEL_CLIENT_ID`, `AIRTEL_CLIENT_SECRET`.

**Expected output:**
Keys in env. Screenshot `day2-airtel-account.png`.

### Task 2: Token exchange

**What to do:**
Create `lib/airtel.js`:

```javascript
import axios from "axios";

let cachedToken = null;
let tokenExpiry = 0;

async function getToken() {
  if (cachedToken && Date.now() < tokenExpiry) return cachedToken;

  const res = await axios.post(
    "https://openapiuat.airtel.africa/auth/oauth2/token",
    {
      client_id: process.env.AIRTEL_CLIENT_ID,
      client_secret: process.env.AIRTEL_CLIENT_SECRET,
      grant_type: "client_credentials",
    }
  );

  cachedToken = res.data.access_token;
  tokenExpiry = Date.now() + (res.data.expires_in - 60) * 1000;
  return cachedToken;
}

export { getToken };
```

Hand-type this. Test with a small script.

**Expected output:**
Token exchange works. Token cached for reuse.

### Task 3: Initiate collection

**What to do:**
```javascript
export async function initiateCollection({ phone, amountKsh, orderId }) {
  const token = await getToken();
  const res = await axios.post(
    "https://openapiuat.airtel.africa/merchant/v1/payments/",
    {
      reference: orderId,
      subscriber: {
        country: "KE",
        currency: "KES",
        msisdn: phone,
      },
      transaction: {
        amount: amountKsh,
        country: "KE",
        currency: "KES",
        id: orderId,
      },
    },
    {
      headers: {
        Authorization: `Bearer ${token}`,
        "X-Country": "KE",
        "X-Currency": "KES",
      },
    }
  );
  return res.data;
}
```

Hand-type every line. Read the Airtel docs to confirm the shape.

**Expected output:**
Collection initiation returns a response.

### Task 4: Server Action + UI wiring

**What to do:**
Create `app/checkout/airtelAction.js` that wraps `initiateCollection`, persists a reference to the order, and returns a status. Add "Airtel Money" as an option in your PaymentStep.

**Expected output:**
Picking Airtel triggers a real collection request.

### Task 5: Callback handler

**What to do:**
Airtel calls a callback URL when the payment completes. Create `app/api/airtel/callback/route.js`. The shape is different from Daraja -- read the docs. Reconcile the order amount as you did in Week 16 Day 2.

**Expected output:**
Callback updates the order status.

## Stretch Goals (Optional - Extra Credit)

- Add a status query endpoint using Airtel's transaction-status API.
- Handle Airtel's specific error codes with friendly messages.
- Write an integration test with a mocked Airtel response.

## Submission Requirements

- **What to submit:** Repo, `lib/airtel.js`, action, callback, `AI_AUDIT.md`.
- **Deadline:** End of Day 2

## Grading Rubric

| Criteria | Points | Description |
|----------|--------|-------------|
| Airtel account + keys | 10 | Sandbox ready. |
| Token exchange with caching | 20 | Works. |
| Collection initiation | 25 | Real API call. |
| Server Action + UI | 20 | Checkout flow offers Airtel. |
| Callback reconciliation | 20 | Amount validation hand-written. |
| Clean commits | 5 | Conventional messages. |
| **Total** | **100** | |

## Common Mistakes to Avoid

- **Missing the X-Country and X-Currency headers.** Airtel requires them on every request.
- **Treating Airtel like Daraja.** The payload shapes are different. Read the docs.
- **Forgetting token expiry.** Cache with a buffer so you refresh before expiry.

## Resources

- Day 2 reading: [Airtel Money Integration.md](./Airtel%20Money%20Integration.md)
- Week 17 AI boundaries: [../ai.md](../ai.md)
- Airtel developer docs: https://developers.airtel.africa/
