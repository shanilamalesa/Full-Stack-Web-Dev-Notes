# Week 6 - AI Boundaries

**Ratio this week: 65% Manual / 35% AI**
**Habit introduced: "Read the docs before the prompt."**
**Shift from last week: Another 5% AI room. But the new habit is anti-AI in spirit -- always open the real documentation before asking AI anything.**

Async JavaScript is the first topic where students who lean on AI too early never recover. Callbacks, promises, async/await, and fetch are the sedimentary rock of modern web development. Miss them and every later week will feel like wading through mud. So this week we bump AI a little but add a brake: official docs first, every time.

---

## Why The Ratio Moved (And Why The Habit Is A Brake)

Async code fails in weird ways. A `fetch` that works in development but times out in production. A promise that resolves in the wrong order. A callback that silently never fires. AI can spot these bugs sometimes, but AI can also confidently hand you the wrong fix -- because async bugs look alike on the surface.

The MDN documentation for `Promise`, `async`/`await`, and `fetch` are some of the best technical writing on the internet. Opening MDN first and AI second is the habit that keeps you honest this week. If you cannot find something on MDN, ask AI. If you could have found it on MDN and were just lazy, the audit will flag it.

35% is a small increase because by Friday you will be building a Data Dashboard that pulls from a public API. That project needs AI help for UI polish, error handling, and layout. Grant it, but only after the fetch itself is manual and correct.

---

## What You Will Feel This Week

- You will hit the word "promise" and have three different mental models fighting in your head for an hour.
- You will chain `.then()` calls and feel clever, then rewrite them as `await` and feel foolish for not starting there.
- You will forget to `await` something and spend twenty minutes debugging why `user.name` is `undefined` when you logged a clearly-filled object a second ago. Welcome to async.
- The fetch to a public API will return data you did not expect, and you will learn that "reading the docs" means reading the API's docs, not just JavaScript's.
- When the Data Dashboard finally pulls real data and renders real charts, you will understand why people love this stack.

---

## What You MUST Do Manually (65%)

### Day 1 -- The callback pattern
- Write five functions that take a callback. Call them with different callback bodies.
- Write a setTimeout chain of three nested callbacks. Feel the callback hell. Do not refactor yet.
- Write one error-first callback in the Node style: `function(err, result) { ... }`. Know what that means.

### Day 2 -- Promises API
- Create a promise manually with `new Promise((resolve, reject) => { ... })`. Resolve it. Reject it. Handle both.
- Convert your callback chain from Day 1 into a promise chain with `.then()`.
- Use `Promise.all`, `Promise.race`, `Promise.allSettled` on an array of three fake promises. Observe the differences.

### Day 3 -- Async/await
- Convert your promise chain from Day 2 into an `async function` with `await`. Compare three versions: callbacks, promises, async. Which is clearest?
- Write a `try/catch` around an `await` call that you make fail. Observe the control flow.
- Understand that `async` functions always return a promise -- prove it by logging the return value of one.

### Day 4 -- Fetch API and working with JSON
- Call a public API manually with `fetch`. Any one: public weather, exchange rates, a joke API, anything.
- Parse the JSON response. Log it. Access the nested fields.
- Add error handling: what happens when the network fails? What happens when the response is a 404? What happens when the JSON is malformed? Handle each case.

### Day 5 and weekend -- Data Dashboard Project
- Build the first version by hand. Pick a public API. Write the fetch manually. Render the data into the DOM manually (no framework).
- At least one loading state, one error state, one empty state. All three, all manually coded.
- Only after the base dashboard works are you allowed to use AI for polish (layout, styling, chart libraries).

---

## You Must Break Things On Purpose

- Call `.then()` on something that is not a promise. Observe.
- Forget to `await` an async function and use its return value immediately. Observe what you get (a Promise object, not the data).
- Throw inside a `.then()` handler without a `.catch()`. Observe the unhandled rejection warning.
- Fetch a URL that is typed wrong. Read the full error stack.

Async errors often happen far from their cause. Breaking things on purpose teaches you the distance.

---

## What You CAN Use AI For (35%)

Five permitted uses:

1. **Explanations** (ongoing).
2. **Comparisons** (ongoing).
3. **Extending working code** (ongoing from Week 4).
4. **Code review** (ongoing from Week 5).
5. **Architecture sketches** (new this week, limited scope).

### Architecture sketches (the new permission)

For the Data Dashboard, you may ask AI to sketch an architecture *before* you write code -- but only with specific constraints:

> "I am building a small Data Dashboard that pulls from [API name]. It should show a loading state, handle errors, and render a chart. I want to build it in plain JavaScript (no framework). What is the minimal structure I should aim for in one file?"

The word "minimal" is important. You are asking for the smallest thing that could possibly work, not a production-grade template. AI's default answer to "build a dashboard" is a 500-line setup. You want 50 lines. Force it to simplify.

### Good vs bad prompts this week

**Bad:** "Write a fetch for the weather API."
**Good:** "I am using `fetch('https://api.example.com/weather')`. MDN says it returns a Response object. I expected `.json()` to return the data directly but it seems to return another promise. What am I missing?"

**Bad:** "Why is my async code broken?"
**Good:** "I have this code [paste 5 lines]. I expected `user.name` to print after the `await` finishes. Instead it prints `undefined`. I checked that the API does return `{ name: ... }` -- I saw it in the browser. What is going wrong?"

**Bad:** "Build error handling for me."
**Good:** "Right now my fetch only handles success. I want to handle: network failure, 404, and bad JSON. Here is my code [paste]. What three `catch` or `if` blocks should I add and where?"

---

## The Read-The-Docs-First Habit

Before you type a prompt this week, you must have:

1. Opened the MDN page for the function you are using (Promise, fetch, async, etc.).
2. Read the first two sections ("Description" and "Syntax") out loud or in your head.
3. Looked at at least one example on the MDN page.
4. Tried your own version.

Only then, ask AI. In the AI Audit, include the MDN link you opened before each prompt. This is non-negotiable.

Why: MDN is the source of truth. AI is a summariser. Reading the source directly builds the habit of going upstream when you need certainty. Junior engineers ask AI; senior engineers ask the docs; the difference matters on hard days.

---

## The 20-Minute Rule (Still)

Twenty minutes of real effort before asking. Same as Week 5. Next week this becomes a softer rule as React arrives and you start composing larger pieces.

One new addition for this week: **before asking AI a prompt, you must spend at least 5 of your 20 minutes with MDN open**. If your prompt ends up duplicating something that was in the first paragraph of the MDN page, the audit will flag it.

---

## Things AI Is Bad At This Week

- **Event loop timing.** Race conditions, order-of-resolution bugs, subtle `await` placements -- AI will often guess wrong because it cannot run your code. Verify everything with `console.log` in between.
- **Specific API responses.** AI does not know the JSON shape of the public API you are calling. It will invent fields that do not exist. Always `console.log(data)` first, then code against reality.
- **Retries and backoff.** Good retry logic is subtle. AI's default retry is usually naive. Write the first version yourself; ask AI to critique only.
- **CORS errors.** The error message is misleading and AI often chases a wrong fix. Read the actual browser console carefully -- if it says "blocked by CORS", the fix is usually server-side, not in your fetch.

---

## Core Mental Models For This Week

- **A callback is a function you pass so someone can call it later.** That "later" is the whole idea of async.
- **A promise is a placeholder for a value that does not exist yet.** It either resolves (value arrives) or rejects (value failed).
- **`async`/`await` is syntactic sugar over promises.** Same machinery, friendlier shape. An `async` function always returns a promise.
- **`fetch` is a promise-returning function.** It resolves with a Response, not with the body. You need a second `await` to get the JSON.

If async feels confusing, come back to these four. Ninety percent of async bugs are confusions between these four ideas.

---

## This Week's AI Audit Focus

Add a required field to every AI interaction log this week: **Which MDN page did I read first?**

Format:

```markdown
### [Bug or question]
- **MDN read first:** [URL]
- **What I tried in the 20 minutes:** [bulleted list]
- **Prompt used:** [...]
- **What AI said:** [...]
- **What I learned:** [...]
```

Facilitators will check that the MDN link is real, relevant, and comes before the prompt. A missing MDN link on any entry is an automatic mark deduction.

---

## Assessment

Week 6 assessment is a live debug on async code:

- Facilitator gives you a broken async function with three subtle bugs (missing `await`, unhandled rejection, wrong order of `.then()` calls). You have 15 minutes to fix all three. No AI.
- Walk through your Data Dashboard's fetch flow: describe every step from the moment you call fetch to the moment the data renders.
- Facilitator asks: "what happens if the user's network drops in the middle of a fetch?" You answer with the specific code path.
- Explain callbacks, promises, and async/await as three ways of doing the same thing. Name one reason to prefer each.

### Live Rebuild Check

Facilitator deletes your fetch function and asks you to rewrite it from memory including error handling. If you cannot, note which part broke down and come back next week.

---

## One Sentence To Remember

"MDN first, AI second." The docs exist. They are free. They are right. Use them before you reach for the shiny thing on your elbow.
