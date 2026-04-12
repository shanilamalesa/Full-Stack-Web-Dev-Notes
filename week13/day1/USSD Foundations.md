# Week 13, Day 1: USSD Foundations

By the end of today, you will have an Africa's Talking sandbox account with a USSD channel pointed at your local machine through ngrok, and an Express endpoint that responds to `*384*1234#` with a three-option menu that a real phone can dial into. You will not have state persistence yet -- that is tomorrow. Today is about understanding what USSD actually is, how it differs from every other channel you have built for so far, and getting a clean "Hello Kenya" working end to end.

This is the start of the fourth big integration of the Marathon. You have already done M-Pesa (Week 10), WhatsApp (Week 11), and the whole Postgres+auth backbone (Week 12). USSD is the last pillar of the "communication stack" the syllabus promised. After this week you can build for any Kenyan user, whether they have a smartphone or a Nokia 3310.

**Prior-week concepts you will use today:**
- Express, routes, middleware, body-parsing (Week 10)
- ngrok exposing localhost over HTTPS (Week 10, Day 2)
- The webhook pattern -- third party POSTs to your server (Week 11, Day 1)
- Environment variables and `.env` discipline (Week 10, Day 1)
- Postgres CRM from Week 12 (you will connect to it on Day 3)

**Estimated time:** 3-4 hours

---

## The Week Ahead

| Day | What You Build |
|---|---|
| Day 1 (today) | Africa's Talking setup, USSD channel, the "Hello Kenya" menu, the `CON`/`END` protocol. |
| Day 2 | State machines for multi-step menus. Redis session store. Navigation, back buttons, input validation. |
| Day 3 | Connect USSD to the CRM -- look up real customers, create real tickets, send SMS confirmations. |
| Day 4 | Deep menus, timeouts, error recovery, African-language strings, accessibility for feature phones. |
| Day 5 | Week recap + peer coding. |
| Weekend | Dial `*384*CODE#` customer support -- the Project 2B deliverable. |

---

## What USSD Actually Is

USSD stands for Unstructured Supplementary Service Data. That is a forty-year-old telecom acronym and it tells you nothing. Here is the mental model instead.

When you dial `*144#` on your phone to check your Safaricom balance, you are not making a phone call. Your phone sends a short text payload over the same signalling channel that carries "incoming call" notifications. The network routes that payload to a piece of software that runs somewhere inside Safaricom, the software answers with another short text payload, and your phone pops up a menu. You type a number, press OK, the menu's next page appears. The whole interaction lasts up to about three minutes (a "session"), then the channel closes whether you are done or not.

Three things make USSD weird compared to everything you have built so far:

1. **It is synchronous, strictly request-response.** No callbacks, no "we will message you when the payment completes". Every interaction is the user typing something and waiting on a blinking cursor for the reply. If your server takes five seconds, the user sees "Connection problem" and gives up.
2. **The session has a hard timeout -- around 90 to 180 seconds.** After that, the carrier kills the session. Whatever state you have stored, the user will have to start over.
3. **The interface is plain text, approximately 182 characters per screen.** No buttons, no images, no emojis on a lot of phones, no Unicode characters above basic Latin. You think like a 1990s BBS, not like a web designer.

Those constraints are also the reason USSD is the single most reliable channel in East Africa. It works on a Nokia 105 that costs 1500 shillings. It works on 2G in Turkana. It does not need data bundles. A rural cooperative in Nandi can use your service for the price of the cheapest SMS bundle -- nothing else. For the SMEs this Marathon targets, being USSD-reachable is often the difference between having customers and not.

### Why Africa's Talking

To run a USSD service you need a "short code" (the `*384#` kind of number) and a connection to every Kenyan mobile operator (Safaricom, Airtel, Telkom). You are not getting either of those directly from Safaricom as a solo developer. You go through an aggregator.

Africa's Talking (AT) is the dominant aggregator in East Africa. They have the short codes, the carrier deals, the sandbox, and a simple HTTP API: when a user dials your code, AT makes a POST request to a URL you register, and your job is to respond with a plain text body. That is it. The entire USSD protocol on your side boils down to "handle an HTTP POST".

There are alternatives -- Hubtel, Infobip, twilio in some markets -- but in Kenya AT is the standard. The patterns you learn here map one-to-one to any other provider.

### The CON/END Protocol

Every response your server sends back to AT starts with one of two words: `CON` or `END`.

- `CON` means "continue the session, here is the next screen".
- `END` means "close the session, show this final message, hang up".

Example `CON` response:

```
CON Welcome to Jetlink Boda.
1. Request a ride
2. My rides
3. Support
```

Example `END` response:

```
END Your ride has been booked. Rider Peter is 4 minutes away.
```

The user picks a menu option by typing `1`, pressing OK, and AT sends another POST with the input. That is how pagination works -- you keep replying `CON` until you are ready to finish, then you reply `END`. Every POST you get has a session id, so on Day 2 when we introduce Redis you will have a key to hang state on.

That is almost the entire protocol. There are a few more fields -- the phone number, the network code, the service code -- but the two-word prefix is the whole thing. People spend a day confused about this because the AT docs bury it; do not be them. `CON` continues, `END` ends.

---

## Setting Up the Africa's Talking Sandbox

AT gives every new account a free sandbox that behaves like the real thing. You do not pay anything, you do not need a business account, and you can test from a simulator on the AT website or from your own phone number.

### Create a sandbox account

1. Go to https://account.africastalking.com/ and sign up. Use a real email. You will need to verify it.
2. After signing in, there is a dropdown near your username that says "Live" or "Sandbox". Switch it to **Sandbox**. All of today's work is in the sandbox.
3. On the left sidebar, click "USSD" and then "Create Channel".
4. For the service code, choose one in the range AT gives you in the sandbox -- something like `*384*1234#`. Write it down; this is your "URL" for the USSD world.
5. For the callback URL, put a placeholder for now: `https://example.com/ussd`. You will come back and replace this once ngrok is running.

### Get your API key

Under "Settings -> API Key" in the dashboard, click "Generate" and copy the key. This is your AT equivalent of a JWT secret -- anyone with it can send SMS and charge your account. Never commit it.

Add to your project `.env`:

```env
AT_USERNAME=sandbox
AT_API_KEY=your_api_key_here
AT_USSD_CODE=*384*1234#
```

In the sandbox, the username is literally the string `sandbox`. In production it would be your AT account username.

---

## Your First USSD Endpoint

We are going to build this in a fresh folder for clarity today, then fold it into the Week 12 CRM on Day 3 once the shape is solid. If you prefer, you can add it directly to the CRM `server/` folder under a new `routes/ussd.routes.js` -- the only reason I am keeping it separate today is to let you focus without the Postgres noise.

```bash
mkdir -p ~/Code/ussd-hello
cd ~/Code/ussd-hello
npm init -y
npm install express dotenv
npm install --save-dev nodemon
```

Create `index.js`:

```javascript
// index.js
require("dotenv").config();
const express = require("express");

const app = express();

// IMPORTANT: AT posts `application/x-www-form-urlencoded`, NOT JSON.
app.use(express.urlencoded({ extended: false }));

app.post("/ussd", (req, res) => {
  const { sessionId, serviceCode, phoneNumber, text } = req.body;

  console.log("USSD hit:", { sessionId, serviceCode, phoneNumber, text });

  let response = "";

  if (text === "") {
    // First page of the session.
    response = "CON Welcome to Jetlink Boda\n";
    response += "1. Request a ride\n";
    response += "2. My rides\n";
    response += "3. Support";
  } else if (text === "1") {
    response = "END You picked: Request a ride. We will come back to this on Day 2.";
  } else if (text === "2") {
    response = "END You picked: My rides.";
  } else if (text === "3") {
    response = "END You picked: Support.";
  } else {
    response = "END Invalid option. Please try again.";
  }

  res.set("Content-Type", "text/plain");
  res.send(response);
});

const PORT = process.env.PORT || 5001;
app.listen(PORT, () => {
  console.log(`USSD server running on :${PORT}`);
});
```

Run it: `npx nodemon index.js`. You should see `USSD server running on :5001`.

### The four fields AT sends you

Read what the handler destructured from `req.body`:

- **`sessionId`** -- a unique string per session. Same user dials `*384*1234#` twice, you get two different session ids. This is your Redis key on Day 2.
- **`serviceCode`** -- the code the user dialled (`*384*1234#`). You will not use this much because you know which code you registered.
- **`phoneNumber`** -- the customer's number, in international format (`+254712...`). You use this for lookups, SMS replies, and for tying USSD sessions to accounts.
- **`text`** -- the entire user journey concatenated with stars. This is the one that surprises people.

### The `text` field is a breadcrumb trail

When the user first dials `*384*1234#`, AT POSTs with `text = ""`. You reply with `CON` and a menu. The user types `1` and hits OK. AT POSTs again with `text = "1"`. You reply with `CON` and a sub-menu. The user types `2`. AT POSTs again with `text = "1*2"`.

That is the whole session state, delivered on every request. The trail grows as the user navigates deeper. You can tell where the user is by splitting `text` on `*`:

```javascript
const inputs = text.split("*");  // ["1", "2"] after two steps
const currentLevel = inputs.length;
const lastInput = inputs[inputs.length - 1];
```

This is the naive way to do USSD state management, and it works for shallow menus with no expensive lookups. By Day 2 we will move to a proper state machine in Redis because the breadcrumb approach falls apart the moment you need to store anything beyond the menu path (e.g. a customer's name, a list of rides).

But for today -- learn the breadcrumb. It is the model AT's documentation assumes.

### Why `application/x-www-form-urlencoded`

Every other webhook you have dealt with -- M-Pesa, WhatsApp -- sends JSON. AT sends form-urlencoded because USSD originated in a world before JSON was a thing, and AT has never broken backwards compatibility. If you forget `express.urlencoded` middleware and rely on `express.json`, `req.body` will be empty and you will burn twenty minutes wondering why. This is the single most common first-day bug with AT; you just skipped it.

---

## Exposing Localhost to AT with Ngrok

Open a second terminal:

```bash
ngrok http 5001
```

Copy the HTTPS forwarding URL -- something like `https://1234-abcd.ngrok-free.app`. Go back to the AT dashboard, edit your USSD channel, and change the callback URL to:

```
https://1234-abcd.ngrok-free.app/ussd
```

Save. Now dial the code from AT's simulator (there is a "Launch Simulator" button on the USSD channel page). The sandbox simulator is a little web widget that pretends to be a Kenyan feature phone. Type your USSD code, hit "Send", and you should see your menu appear. Pick an option, see the `END` message.

If nothing happens, check:

1. The ngrok tunnel is still open -- they expire after a few hours on the free plan.
2. Your Express server logged a "USSD hit:" line. If it did not, AT never reached you -- usually a wrong callback URL.
3. Your server replied quickly. AT times out after ~10 seconds. If your handler is hanging (say, a stuck `await`), AT shows a generic error on the simulator.

Once the simulator works, try it from your real phone. Dial the code you registered. On the sandbox, AT will actually put the call through for you from the sandbox short code, and your Nokia or Samsung or whatever you have in your pocket will ring through to your ngrok tunnel. This is the moment USSD stops being theoretical.

---

## Designing For 182 Characters

Now that the wires are connected, spend the rest of the session thinking about the thing the syllabus barely mentions: UX on a feature phone.

Rules of thumb:

**Under 182 characters per `CON` response.** Some phones will crop longer responses; all phones will show them cramped. Count newlines and spaces. A menu like:

```
CON Welcome to Jetlink Boda. Please select an option from the list below to continue.
1. Request a new ride in your area
2. View my recent rides and their status
3. Contact support for help
```

is too long. A good one looks like:

```
CON Jetlink Boda
1. Request a ride
2. My rides
3. Support
```

**No more than five options per menu.** Users have to type the number and read the label. On a small screen with no scrolling, six options will spill off the bottom.

**Numbers as labels, not verbs.** "1. Cancel" is clear. "Press 1 to cancel" wastes characters and reads worse because the user is already used to "number dot label".

**Always leave an exit.** Every menu below the root should have a "0. Back" or "00. Main menu" option. Without it, a lost user hangs up and re-dials, which costs them time and annoys them. This is true etiquette for USSD -- feature-phone users cannot swipe back.

**Default to plain ASCII.** Accented characters and emojis render as `?` on many phones. "Nairobi" is safe; "Nairóbi" is not. You can localise later using ASCII-only transliterations.

**Pre-compute. Do not make the user wait.** If a menu needs data from your database, fetch it fast. Anything above ~2 seconds is a bad experience; anything above ~5 is a dropped session. This is why Day 2 introduces Redis -- you cache the things the menu needs, so the live USSD path is never waiting on Postgres for the third time in a session.

### A deliberate redesign

Look at your Day-1 hello menu. Is it under 182 characters? (Yes, 52.) Does it have five or fewer options? (Yes, 3.) Is it all ASCII? (Yes.) Does it tell the user what they are inside of? (Yes -- "Jetlink Boda" at the top.) Does a top-level menu need a "Back" option? (No, because there is nothing above it.)

Good. Now look at your `END` messages. The one for "Request a ride" says "We will come back to this on Day 2." That is fine as a placeholder, but notice the shape: a good `END` message tells the user *what happened* and *what comes next*. "Your ride has been booked. Rider Peter will arrive in 4 minutes. Cost: KSh 180. You will receive an SMS with details." Four sentences, 114 characters, and the user walks away with everything they need.

Write better `END` messages than you think you need to. Feature-phone users will remember the one they saw, because they cannot scroll back through a chat history to check.

---

## Checkpoint

Before stopping, prove each of these.

1. You can dial `*384*1234#` from the AT simulator and see your menu.
2. You can pick each of the three options and see a matching `END` message.
3. Your Express server prints a "USSD hit:" log on every POST.
4. You can dial the code from a real Kenyan number (your own phone, if you have one) and see the menu.
5. You can explain the difference between `CON` and `END` in one sentence without looking.
6. You can explain what `text` contains on the *third* POST of a session where the user typed `1`, then `2`, then `3`. (Answer: `"1*2*3"`.)
7. Your `.env` has `AT_API_KEY` but it is not in `git`.

Commit:

```bash
git init
echo "node_modules/\n.env" > .gitignore
git add .
git commit -m "feat: ussd hello menu via africas talking sandbox"
```

---

## What To Read Before Tomorrow

- Africa's Talking USSD docs: https://developers.africastalking.com/docs/ussd/overview -- skim the "building a USSD app" section.
- The Redis quickstart: https://redis.io/docs/latest/develop/get-started/ -- just the first page, to get a feel for the commands.
- Optional: read how Safaricom's `*144#` menu is laid out. Pay attention to how their top menu is exactly four options and every sub-menu has a "Back" on 0. That is the standard to aim for.

Tomorrow we replace the if/else chain with a real state machine, introduce Redis for session storage, and build a three-level menu with form-like prompts ("type your name", "type the amount").
