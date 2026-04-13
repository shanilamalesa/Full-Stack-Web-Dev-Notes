# Week 10, Day 1: Introduction to Payments

> **AI boundaries this week:** 50% manual / 50% AI. Habit: *Never trust AI with money -- every line that touches an amount is hand-written and hand-reviewed.* See [ai.md](../ai.md).

These are supplementary reading notes to accompany **The Backend Foundation** tutorial. Read these before or alongside the hands-on work. They give you the context behind what you are building and why it matters -- not just as a coding exercise, but as a tool that solves a real problem for real people.

**Estimated reading time:** 15-20 minutes

---

## Why Payments Matter to You as a Developer

Every business, at its core, does one thing: it exchanges value for money. A web designer builds a website and needs to get paid. A mama mboga sells tomatoes and needs to collect payment. A SaaS company charges a monthly subscription. The mechanism changes, but the fundamental transaction is the same.

As a developer, the moment you understand how to move money programmatically, you become significantly more valuable. Most software exists to either save money, make money, or move money. The developer who can integrate payments into an application is the developer who ships products that generate revenue. That is not a small thing. A beautiful frontend that cannot collect payment is a brochure. A backend that can process transactions is a business.

This week you are building a payment link system. But the real skill you are acquiring is the ability to integrate with financial APIs. Once you understand the pattern -- authenticate, initiate a transaction, handle the async result -- you can apply it to Stripe, PayPal, Flutterwave, Paystack, or any payment provider in the world. The specifics change. The architecture does not.

---

## A Brief History of Payments

### Before Digital: Cash and Barter

For most of human history, payments were physical. You handed someone coins, notes, or goods. This worked for face-to-face transactions but had obvious limitations. You could not pay someone in another city without physically transporting money. You could not buy something at 2 AM. You needed exact change or trusted the other party to give you the right amount back.

### The Card Era

Credit and debit cards digitized the payment for the first time. Instead of handing over cash, you presented a plastic card. The card network (Visa, Mastercard) acted as a middleman -- it verified the cardholder, moved money from the buyer's bank to the seller's bank, and took a small fee for the service.

Cards were revolutionary in Europe and North America. But they required infrastructure: point-of-sale terminals, reliable internet connections, bank accounts for both parties, and a postal system to deliver the cards. In much of Africa, that infrastructure did not exist at scale. As of 2023, fewer than 10% of adults in sub-Saharan Africa had a credit card. Cards solved the payment problem for the developed world, but they skipped most of the continent.

### The Mobile Money Revolution

In 2007, Safaricom launched M-Pesa in Kenya. The idea was simple: use the mobile phone -- which people already had -- as a financial tool. No bank account required. No plastic card. No internet connection (M-Pesa runs on USSD, which works on the most basic feature phones). Just a SIM card and a PIN.

M-Pesa did not try to replicate the Western banking model. It built something new from the ground up, designed for the realities of the Kenyan market: low bank penetration, high mobile phone adoption, a large unbanked population that still needed to send and receive money.

Within two years of launch, M-Pesa had more users than all Kenyan banks combined. Today, M-Pesa processes over 60 billion KES in transactions daily. It operates in Kenya, Tanzania, Mozambique, DRC, Lesotho, Ghana, Egypt, and is expanding further. It has been studied and replicated across Asia and Latin America.

This was not just a product launch. It was a fundamental shift in how an entire economy moves money.

---

## How M-Pesa Actually Works

To build on top of M-Pesa, you need to understand what happens under the hood. Not at the network protocol level, but at the level that matters for your code.

### The Basics

Every M-Pesa user has a mobile money account tied to their phone number. This account is not a bank account -- it is held by Safaricom, which is a mobile network operator, not a bank (though it is regulated by the Central Bank of Kenya). The user deposits cash at an M-Pesa agent (those green kiosks you see everywhere), and that cash becomes a digital balance on their phone.

From there, the user can:
- **Send money** to another phone number (Person to Person, or P2P)
- **Pay a bill** to a business paybill number (Customer to Business, or C2B)
- **Buy goods** at a till number
- **Withdraw cash** at an agent

All of this happens through USSD menus -- those text-based screens you get when you dial `*334#`. No smartphone required. No internet required. Just a basic phone with a SIM card.

### The Business Side: Paybill and Till

When a business wants to receive M-Pesa payments, it registers for either a **paybill number** or a **till number** with Safaricom.

A **paybill number** is like a business bank account number. Customers send money to the paybill and include an account reference (like an invoice number) so the business knows which payment is for which customer. Paybill numbers are used by larger businesses, utilities (KPLC, Nairobi Water), and online platforms.

A **till number** is simpler. Customers just pay to the till number, no reference needed. This is what you see at the counter of your local duka or supermarket.

For this project, you are working with a paybill (shortcode 174379 in the sandbox). The "account reference" is your payment link ID, so when the payment arrives, you know exactly which invoice it belongs to.

### STK Push: The Developer's Entry Point

In the early days, M-Pesa payments were entirely user-initiated. The customer had to open the USSD menu, navigate to "Lipa Na M-Pesa", type in the paybill number, type in the account reference, type in the amount, and enter their PIN. That is a lot of steps, and a lot of room for the customer to type the wrong number.

STK Push changed this. With STK Push, the business initiates the payment. Your server sends a request to Safaricom saying "ask this phone number to pay this amount." Safaricom then pushes a prompt directly to the customer's phone. The customer sees a single popup: the business name, the amount, and a field to enter their PIN. One step instead of five. Fewer errors. Higher conversion rates.

This is the mechanism you are implementing in your capstone project. When the client clicks "Pay" on your payment page, your backend triggers an STK push. The client's phone buzzes with the payment prompt. They enter their PIN. Done.

### The Daraja API

Daraja (Swahili for "bridge") is Safaricom's developer platform. It is the API layer that sits between your application and M-Pesa's core systems. Through Daraja, you can:

- **Initiate STK Push payments** (what you are building)
- **Register callback URLs** to receive payment notifications
- **Check transaction status** for reconciliation
- **Reverse transactions** if something goes wrong
- **Query account balances**
- **Initiate Business-to-Customer payments** (sending money to a phone number)

Daraja provides a sandbox environment where you can test everything without real money. The sandbox uses test credentials and a test shortcode (174379). When you are ready to go live, you apply for production credentials through Safaricom's business portal, and you swap the sandbox URL for the production URL. The code stays the same.

---

## The Payment Flow in Your Application

Your capstone project implements what the fintech industry calls a **payment gateway** -- a system that sits between the customer and the payment provider, handling the transaction flow.

Here is the flow mapped to the components you are building:

```
CLIENT (browser)                YOUR SERVER              SAFARICOM
     |                              |                        |
     |  1. Clicks "Pay"             |                        |
     |------ POST /api/pay -------->|                        |
     |                              |  2. Authenticate       |
     |                              |------- OAuth --------->|
     |                              |<------ Token ----------|
     |                              |                        |
     |                              |  3. STK Push request   |
     |                              |------- POST ---------->|
     |                              |<------ Accepted -------|
     |  4. "Check your phone"       |                        |
     |<----- 200 OK ----------------|                        |
     |                              |                        |
     |                  5. Phone buzzes, user enters PIN      |
     |                              |                        |
     |                              |  6. Callback (webhook) |
     |                              |<------ POST -----------|
     |                              |------- 200 OK -------->|
     |                              |                        |
     |                              |  7. Update database    |
     |                              |  8. Generate receipt   |
     |                              |                        |
     |  9. Poll: "Is it paid?"      |                        |
     |------ GET /status ---------->|                        |
     |<----- { status: "paid" } ----|                        |
     |                              |                        |
     | 10. Show success + receipt   |                        |
```

Notice the gap between steps 4 and 6. After your server sends the STK push request and Safaricom accepts it, there is a period of waiting. The customer might take 5 seconds to enter their PIN, or 30 seconds, or they might walk away and never complete it. Your server does not sit idle during this time -- it has already responded to the client's browser (step 4) and is handling other requests. When Safaricom eventually sends the callback (step 6), your server processes it asynchronously.

This is the fundamental nature of payment systems: they are asynchronous. You initiate a transaction, and the result arrives later through a separate channel. Every payment provider works this way -- Stripe, PayPal, Flutterwave, all of them. The mechanism differs (webhooks, polling, long-polling, WebSockets), but the asynchronous pattern is universal.

---

## Mobile Money Beyond Kenya

M-Pesa is the most well-known mobile money platform, but it is far from the only one. Mobile money has spread across Africa and into parts of Asia because it solves a real problem: financial access for people without traditional bank accounts.

**Tanzania** -- Vodacom M-Pesa, Airtel Money, Tigo Pesa (now MobiPay). Tanzania has multiple competing mobile money providers, and interoperability between them is a major challenge.

**Uganda** -- MTN Mobile Money, Airtel Money. Uganda's mobile money market is one of the most active in East Africa, with mobile money transactions exceeding the country's GDP.

**Ghana** -- MTN MoMo, Vodafone Cash, AirtelTigo Money. Ghana was one of the first countries to achieve mobile money interoperability, letting users send money across different providers.

**Nigeria** -- OPay, PalmPay, and bank-led mobile money platforms. Nigeria's market is different because the Central Bank initially restricted mobile money to banks, slowing adoption compared to East Africa. This opened the door for fintech startups like Paystack (acquired by Stripe) and Flutterwave.

**India** -- UPI (Unified Payments Interface) took a different approach: instead of mobile money wallets, India built a real-time bank-to-bank transfer system accessible through apps like Google Pay, PhonePe, and Paytm. UPI processes over 10 billion transactions per month.

**The Philippines** -- GCash and Maya (formerly PayMaya) dominate. GCash alone has over 80 million users in a country of 115 million.

The patterns you learn building with M-Pesa's Daraja API transfer directly to these platforms. The auth mechanism might differ. The payload fields will have different names. But the flow -- authenticate, initiate payment, handle async callback -- is the same everywhere.

---

## Key Concepts You Will Encounter

These terms come up repeatedly in payment systems. Understanding them now will make the technical implementation on Day 2 much clearer.

### Transaction vs. Payment

A **payment** is the act of transferring money from one party to another. A **transaction** is the record of that payment. When a customer pays KES 5,000, one payment happens, but multiple transaction records might be created: the debit from the customer's M-Pesa account, the credit to the business's paybill, the fee deduction, and the reconciliation entry. In your database, you store the transaction record, not the payment itself (the actual money movement happens inside Safaricom's systems).

### Idempotency

If Safaricom sends the same callback twice (because of a network glitch or timeout), your server should not process the payment twice. This is called **idempotency** -- performing the same operation multiple times produces the same result as performing it once. In your code, you handle this by checking if a payment is already marked as "completed" before processing the callback. This is not theoretical. Duplicate callbacks happen in production. Systems that do not handle them charge customers twice.

### Reconciliation

At the end of each day (or each week, or each month), a business needs to verify that the payments they received match the transactions recorded in their system. This is **reconciliation**. It answers the question: "Did KES 500,000 in M-Pesa payments actually arrive in our paybill, and do those match the 500,000 we recorded in our database?" This is why you store the `raw_callback` JSON in your payments table -- the raw data from Safaricom is your source of truth if there is ever a discrepancy.

### Settlement

When a customer pays to your M-Pesa paybill, the money does not instantly appear in your bank account. It sits in your paybill's M-Pesa float. **Settlement** is the process of moving that money from your M-Pesa float to your actual bank account. For most businesses, this happens automatically on a schedule (daily or weekly) configured with Safaricom. Your application does not handle settlement -- Safaricom does. But knowing the term matters because your clients will ask about it.

### Sandbox vs. Production

The Daraja **sandbox** is a testing environment where no real money moves. You use test credentials, a test shortcode (174379), and test phone numbers. Everything works the same as production, but the transactions are simulated. When your application is ready for real users, you apply for **production** credentials through Safaricom's business portal. The only code change is swapping the base URL from `sandbox.safaricom.co.ke` to `api.safaricom.co.ke` and updating your credentials.

---

## Why This Project Matters for Your Career

### The African Fintech Market

Africa's fintech sector is the fastest-growing in the world. Between 2019 and 2023, African fintech startups raised over $6 billion in venture funding. Companies like M-Pesa, Flutterwave, Paystack, Chipper Cash, and Wave are building the financial infrastructure for a continent where 57% of adults are still unbanked or underbanked.

This is not a mature, slow-moving industry. It is growing fast, and the demand for developers who understand payment systems far exceeds the supply. A developer in Nairobi who can integrate M-Pesa, handle webhooks, and build reliable transaction processing has a skill set that companies are actively hiring for -- both locally and internationally.

### What Companies Want

When a fintech company or a business that accepts digital payments hires a developer, they are looking for someone who understands:

- How to authenticate with third-party APIs (you are learning this with OAuth on Day 2)
- How to handle asynchronous workflows where the result comes later (the STK push callback flow)
- How to store transaction data reliably and handle edge cases (duplicate callbacks, failed payments, timeouts)
- How to build user-facing payment interfaces that are simple and trustworthy (the React payment page on Day 3)

By the end of this week, you will have demonstrated all four.

### Beyond This Project

The payment link system you are building is a real product category. Companies like Pesapal, IntaSend, and Kopokopo in Kenya offer similar tools. Stripe has Stripe Payment Links. PayPal has PayPal.Me. You are building a simplified version of what these companies sell as a service.

After this capstone, you could extend it into a real product: add multiple payment methods (card payments via Flutterwave or Paystack alongside M-Pesa), add recurring payments for subscriptions, add a merchant dashboard with analytics, or add multi-currency support for cross-border payments. Each of these is a natural extension of what you build this week.

---

## Reading Checkpoint

Before you continue to the hands-on backend tutorial, make sure you can answer these questions:

1. What problem does M-Pesa solve that traditional banking did not address in Kenya?
2. What is the difference between a paybill number and a till number?
3. What is STK Push, and why is it better than the customer manually entering payment details through USSD?
4. Why are payment flows asynchronous? Why can your server not wait for the payment result in the same HTTP response?
5. What is idempotency, and why does it matter when handling payment callbacks?
6. What is the Daraja API, and what is the difference between sandbox and production?

If you can answer those six questions, you have the context you need. Open **The Backend Foundation** notes and start building.
