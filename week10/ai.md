# Week 10 - AI Boundaries

**Ratio this week: 50% Manual / 50% AI**
**Habit introduced: "Never trust AI with money."**
**Shift from last week: Same ratio, new rule. This week touches real money for the first time. Every line that moves a shilling is audited manually.**

Payments are where AI can cause actual harm. A misplaced decimal, a missed idempotency key, a swallowed error, a refund to the wrong party -- all real incidents at real companies, all easy to generate with a fast AI prompt. This week you build a working M-Pesa Paylink and Receipt system, and you do it with a new, permanent discipline: **security and money code is never left as AI-shaped**.

AI can write the scaffolding. You audit the payment logic.

---

## Why The Ratio Is The Same, But The Rule Is Different

You are at 50/50 in skill, which means AI can realistically write half your code without slowing you down. But "half" is not evenly distributed. Some things AI writes well (React forms, PDF generation, UI polish, routing, dashboards). Some things AI writes dangerously (signature verification, callback idempotency, amount validation, receipt numbering).

This week you learn to split your work into those two buckets explicitly, and to treat every line in the dangerous bucket as if a senior engineer were reviewing it behind your shoulder. The ratio stays, but the attention is uneven.

---

## What You Will Feel This Week

- The first successful STK push will be the most exciting moment in the programme so far.
- You will test with small amounts (KSh 1, KSh 10) and still feel nervous every time your phone buzzes with the prompt.
- You will build a PDF receipt and feel like you are making a real product.
- Reading Safaricom's Daraja docs will feel like reading a tax form. That is correct. Daraja is a tax-form API.
- You will be tempted to skip reading the callback structure because "AI knows it". Do not. Callback shapes are where bugs live.

---

## What You MUST Do Manually (50%)

### Day 1 -- Intro to payments + backend foundations
- Read the Daraja documentation yourself. Not a summary. The real pages for: Authorization, STK Push, C2B Callback, Transaction Status.
- Set up an Express backend manually. You have done this before (Week 6, via pattern) -- now repeat it, refresh it, and make sure you remember why each middleware is there.
- Environment variables: `MPESA_CONSUMER_KEY`, `MPESA_CONSUMER_SECRET`, `MPESA_SHORTCODE`, `MPESA_PASSKEY`, `CALLBACK_URL`. Every variable typed by you, never committed.

### Day 2 -- M-Pesa STK Push (MANUAL ZONE)
- Token generation: write the OAuth dance yourself. Observe the base64 encoding, the token expiry, the response shape.
- STK Push initiator: write the request body yourself using Daraja's exact field names. Every field, every casing, by hand.
- Callback handler: write the route that receives Safaricom's POST yourself. Log the full payload. Read it. Understand every key.
- Amount validation: **manual, non-negotiable**. Every callback must verify that the amount Safaricom reports matches the amount you asked for. If they disagree, you log the discrepancy and flag the order for review. This is the "never trust AI with money" rule in its hardest form.
- Idempotency: manual. Every callback must handle being called twice with the same `CheckoutRequestID` without double-processing. Write it by hand.

### Day 3 -- PDF receipts + React frontend
- Install pdfkit (or pdf-lib) yourself. Read its docs.
- Generate a receipt PDF with the transaction details. Make it look decent -- header, amount, reference, date.
- Wire up the React frontend: a form that takes phone number and amount, triggers the STK push, polls for completion, shows the receipt.

### Day 4 -- End-to-end testing with sandbox
- Test the full flow with Safaricom's sandbox test numbers. Handle the success case, the user-cancelled case, the timeout case, and the wrong-PIN case.
- Log every callback to a file or database for inspection.
- Prove the idempotency works by manually re-POSTing a callback with the same ID and confirming nothing changes.

### Day 5 and weekend -- Harden
- Add error UI: loading state, error messages, retry button. This part AI can scaffold.
- Deploy the backend somewhere (Railway, Render, or ngrok for testing). Ensure the callback URL is reachable from Safaricom's network.

---

## The Money Rule (Hardest Habit So Far)

Every line of code that:
- moves money,
- validates amounts,
- handles refunds,
- stores transaction references,
- verifies signatures or webhooks,
- updates balances,

must be written by you, reviewed line-by-line, and explained verbally to yourself or a rubber duck before commit. If AI wrote any of those lines, you rewrite them (or at least re-type them) and make sure you can defend every character.

This rule holds for the rest of the programme -- Weeks 17, 18, 19, 22, 28, 29. Learn it now.

---

## You Must Break Things On Purpose

- Send an STK push with an amount of 0 or negative. Observe Daraja's response.
- POST a fake callback to your endpoint with a wrong reference. Observe your idempotency/validation.
- Submit a callback with a different amount than the push. Your amount validation must catch it.
- Remove the idempotency check temporarily. Fire two identical callbacks. Observe the double-processing. Add it back.

These "mistakes" are the shape of real incidents. Practise them now.

---

## What You CAN Use AI For (50%)

Permitted uses:

1. All prior permissions.
2. **UI scaffolding** for the frontend -- forms, loaders, receipt previews.
3. **PDF generation boilerplate** -- the pdfkit API is verbose and AI helps with layout.
4. **Explaining Daraja error codes** after you have tried to understand them yourself.
5. **Code review of your payment logic** -- "here is my callback handler, what edge cases am I missing?"

**AI is forbidden for:**

- Writing the amount validation logic.
- Writing the idempotency check.
- Writing the signature verification (when we add it in Week 18).
- Writing the token refresh or expiry handling.
- Deciding when to mark an order paid.

### Good vs bad prompts this week

**Bad:** "Integrate M-Pesa for me."
**Good:** "Here is my STK push function [paste]. It works on success but I am not sure about my error handling -- what error cases from Daraja should I explicitly handle, and which can I catch in a generic block?"

**Bad:** "Write my callback handler."
**Good:** "My callback handler currently marks the order paid on any ResultCode 0. I know I also need idempotency. What is the standard pattern to make this idempotent in Express with a SQL database? I will write the implementation -- I just want the pattern."

**Bad:** "Fix my amount validation."
**Good:** "I wrote amount validation: `if (callback.Amount !== expected) { return; }`. Is there a subtlety with Daraja's amount format I am missing -- do they send it as a string, a number, or wrapped in something else?"

---

## The 30-Minute Rule

This week only: before asking AI about anything money-related, you must spend at least **thirty minutes** trying yourself. Longer than previous weeks. Reason: money bugs are subtle, and rushed AI questions in payment code cause the worst failures.

Outside of money code, the 25-minute rule from Week 8 still applies.

---

## Things AI Is Bad At This Week

- **Daraja's quirks.** Daraja returns different shapes in different sandboxes. AI's training data has outdated Daraja responses mixed with current ones. Trust the docs over AI.
- **Real-world failure modes.** "What happens when the user's phone dies mid-PIN?" is a real question. AI's answer is usually too clean. Test it yourself.
- **Idempotency keys.** AI's default idempotency is often "check if the record exists", which misses the race condition. Real idempotency uses a unique key constraint in the DB. Demand that pattern specifically.
- **Receipt number uniqueness.** AI will generate random IDs that can collide. Use the DB's auto-incrementing ID or a proper UUID library. Never roll your own.
- **Cents vs shillings.** Always work in the smallest unit (cents). AI sometimes silently switches. Check every arithmetic operation.

---

## Core Mental Models For This Week

- **A payment is a distributed transaction.** Two parties (you and Safaricom) need to agree on its outcome. That agreement is why callbacks and idempotency exist.
- **A callback is an authenticated promise.** Safaricom promises to call you. You verify the call is real and process it exactly once.
- **Money must reconcile.** At any moment, the sum of what your database thinks you have should match the sum of what Safaricom thinks you received. Build with reconciliation in mind from day one.
- **A receipt is a legal artefact.** Customers use them for tax, refunds, and disputes. Never generate one for a transaction that did not settle.

---

## This Week's AI Audit Focus

Add a required section: **Money code audit.** List every line of code that touches money and classify it:

```markdown
### File: routes/mpesa.js

| Line range | Touches money? | Who wrote it | Reviewed line-by-line? |
|---|---|---|---|
| 10-25 | Yes (amount validation) | Me | Yes |
| 27-45 | Yes (callback handler) | Me + AI review | Yes |
| 47-60 | No (logging helper) | AI scaffolded | No (not money) |
```

Every "Yes" line must have "Me" or "Me + AI review" in the third column. If anything in the third column says "AI generated, not reviewed", you fail this week.

---

## Assessment

Week 10 assessment is a code walkthrough focused on money:

- Walk through your STK push -> callback -> receipt flow. Explain every line that touches an amount.
- Live test: facilitator sends a fake callback with a wrong amount to your endpoint. Prove your validation catches it.
- Live test: facilitator sends the same callback twice. Prove your idempotency catches it.
- Explain the OAuth token refresh flow. No AI allowed.
- Explain why you use cents instead of shillings in your code.

### Live Rebuild Check

Facilitator deletes your amount validation and asks you to rewrite it from memory. This one is critical -- if you cannot, you do not pass until you can.

---

## One Sentence To Remember

"Money code is the kind of code where 'it works in the happy path' is not enough." Every edge case matters. Every edge case is on you.
