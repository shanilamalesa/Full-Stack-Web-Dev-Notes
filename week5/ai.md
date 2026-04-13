# Week 5 - AI Boundaries

**Ratio this week: 70% Manual / 30% AI**
**Habit introduced: "AI as code reviewer."**
**Shift from last week: You start pasting your code to AI and asking it to critique -- not to write.**

This week you move from writing individual functions to organising them. Objects, classes, prototypes, inheritance, ES6 modules. This is how real codebases are shaped. The week also hosts your first proper cohort project -- something you will demo to your peers.

The new habit is a role reversal: last week AI extended your code; this week AI reviews it. You paste your working code, you ask AI "what would you change and why?", and you evaluate the suggestions with your own judgement. You may accept zero of them. That is fine. The point is the review conversation, not the rewrite.

---

## Why The Ratio Moved

OOP is tricky to learn from AI. Every AI tutorial on classes is too clean. Real OOP learning happens when you build a thing the wrong way, feel the pain of the wrong shape, refactor, and remember the feeling. Let AI hand you a perfect UserClass and you will never feel the pain -- which means you will never learn why encapsulation matters.

30% is the ceiling because by the weekend you need AI to help you polish the cohort project. Not for originating the ideas. For polishing and review.

---

## What You Will Feel This Week

- You will ask "why do I need a class when I can just use an object?" The answer will arrive in the form of a bug that a class would have prevented.
- Prototype chain will feel like wizardry for two days, then become obvious.
- You will copy a class pattern from Week 4's arrow functions and mix `this` confusion into it. That is normal.
- ES6 modules will feel like organisation at last. You will finally put each class in its own file and feel adult about it.
- The cohort project will force decisions: "does this belong on the User class or the Cart class?" There is no perfect answer. Pick one and defend it.

---

## What You MUST Do Manually (70%)

### Day 1 -- Object literals and property access
- Write 10 object literals with different property types (strings, numbers, arrays, nested objects, method shorthand).
- Access properties with dot notation, bracket notation, destructuring. Use each at least three times.
- Update properties, delete properties, check for existence with `in` and `hasOwnProperty`.

### Day 2 -- Constructor functions and ES6 classes
- Write one constructor function the "old school" way (`function User(name) { this.name = name; }`).
- Rewrite the same thing as an ES6 class. Compare both files. Which is clearer?
- Write a class with at least one static method, one instance method, one getter, one setter. Use all four.
- Instantiate your class with `new`. Make at least three instances with different data.

### Day 3 -- Prototype chain and inheritance
- Extend a class: `class Admin extends User`. Override one method. Call `super()` in the constructor.
- Explore `__proto__` manually in the browser console. Trace the chain from your instance all the way up to `Object.prototype`. Draw it on paper.
- Explain in one sentence what inheritance buys you that plain composition does not. (Honest answer: not always much. Note your honest opinion.)

### Day 4 -- ES6 modules
- Split the classes you wrote into separate files. Each file exports one thing.
- Import them into a main file. Use both default and named exports.
- Understand why `import` statements are at the top -- try moving one and observe the error.

### Day 5 and weekend -- Cohort project
- Build the cohort project manually for the first two-thirds of your implementation time. No AI originating architecture.
- Organise your code into classes and modules that make sense. Write a short `README.md` explaining the class structure.
- Only in the last third do you allow AI as a code reviewer.

---

## You Must Break Things On Purpose

- Forget `new` when instantiating a class. See the error.
- Override a method and forget to call `super()` from the subclass constructor. See the error.
- Export a class as default in one file and try to import it as a named import elsewhere. See the error.
- Create a circular import between two modules. See what happens.

Each error message teaches you a specific shape. Collect them.

---

## What You CAN Use AI For (30%)

Four permitted uses this week:

1. **Explanations** (ongoing).
2. **Comparisons** (ongoing).
3. **Extending working code** (ongoing from Week 4).
4. **Code review** (new this week).

### The code review pattern

You finish a class. It works. You paste it to AI and ask:

> "Here is my `Cart` class [paste]. It handles adding items, removing items, and calculating totals. Read it as a code reviewer. What would you change, and why? Do not rewrite -- just list the issues."

The "do not rewrite -- just list the issues" part is crucial. You want the review, not the rewrite. Otherwise AI will give you a shiny new version and you will learn nothing.

Read the list. For each item, decide:
- Do I agree?
- If I disagree, why?
- Is this a taste difference or a real bug?
- If I accept the change, what exactly am I changing and why?

Write your decisions in the AI Audit. This is the week you start building engineering taste.

### Good vs bad prompts this week

**Bad:** "Write a User class for me."
**Good:** "Here is my User class [paste]. It works. What are three things a senior developer would critique about it?"

**Bad:** "Fix my inheritance."
**Good:** "My `Admin extends User` class calls methods that do not exist on `User` [paste]. I expected inheritance to somehow make them work. What am I misunderstanding about how `extends` works?"

**Bad:** "Make my project better."
**Good:** "Here is my cohort project folder structure [paste the tree]. Each class is in its own file. Is this a reasonable way to organise a small OOP project, or would you split things differently?"

---

## The Code Reviewer Habit

When you ask AI to review, demand specific feedback. Generic "looks good" replies are useless. Force specificity:

- "What is the single biggest issue?"
- "What naming decision would you change?"
- "Where is my encapsulation leaking?"
- "Is there a place where I used inheritance when composition would be cleaner?"

You are training AI (and yourself) to have opinions. An opinion you disagree with is more valuable than an agreement.

### When AI reviews wrong

AI will sometimes flag non-issues. A classic example: "I would consider making this method private." In JavaScript, private methods are a newer feature (`#method`) and sometimes AI will suggest it where it makes no practical sense for your tiny project. **You are allowed to say no.** Write "AI suggested X, I disagreed because Y" in your audit. Disagreement is progress.

---

## The 20-Minute Rule

Before asking AI anything this week, spend at least **twenty minutes** of real effort. Longer than Week 4's fifteen minutes. The reason: OOP bugs are usually bugs in your mental model, and mental model bugs need more staring to find.

Twenty minutes of:
1. Drawing the class relationships on paper.
2. Running `console.log(this)` inside the methods you suspect.
3. Re-reading the MDN page for `class`, `extends`, or `this`.
4. Talking to the rubber duck.
5. Trying a smaller version of the same problem.

---

## Things AI Is Bad At This Week

- **Encapsulation decisions.** "Should this live on the class or outside?" is a design question. AI will usually leave it where you put it unless you ask explicitly. You need to ask.
- **"Clean code" vs "clever code".** AI leans clever when you give it freedom. Clever code impresses nobody in code review. Ask for clean, not clever.
- **Your cohort project's shape.** AI does not know what your teammates are working on. Integration-shaped bugs ("my class expects an object; the other team's function returns an array") will not show up until you run them together.
- **Module cycles.** Circular imports are a classic that AI sometimes makes worse. Always verify module structure manually when you split files.

---

## Core Mental Models For This Week

- **An object is a bag of related data.** A loose collection, flexible but easy to misuse.
- **A class is a factory for objects of the same shape.** It enforces structure. Less flexible, less error-prone.
- **A prototype is a shared parent object.** Every instance of the same class points at the same prototype. That is how methods are shared.
- **A module is a file with an interface.** Exports are the interface; imports are the contract.

If a design question feels hard, walk through these four. Almost every week-5 decision reduces to one of them.

---

## This Week's AI Audit Focus

Add a required section to your audit: **Code Review Decisions.**

For every AI code review you ran, record:

```markdown
### [Class or file name]
- **What I asked:** [the review prompt]
- **Issues AI raised:** [bulleted list]
- **Accepted:** [which ones and why]
- **Rejected:** [which ones and why]
- **What I learned about my own code:** [1-2 sentences]
```

Facilitators grade this section heavily. An audit with all "accepted" or all "rejected" is a red flag -- it means you are not reading critically.

---

## Assessment

Week 5 assessment is a class design review:

- Walk through your cohort project class structure on a whiteboard.
- Pick one class and explain every method: what it does, why it exists, why it is on this class instead of another.
- Live task: the facilitator asks "add a new feature that needs a new method -- where does it go?" You answer in 60 seconds with reasoning.
- Explain inheritance vs composition in your own words. One example of each.
- Show one AI code review decision and defend it.

### Live Rebuild Check

Facilitator deletes one method from one of your classes. You rewrite it from memory. If you cannot, note it and come back next week with proof.

---

## One Sentence To Remember

"AI reviews. You decide." The conversation matters more than the suggestion. Agreement means nothing without a reason; disagreement is evidence you have taste.
