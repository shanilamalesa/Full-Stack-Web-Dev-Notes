# Week 2 - AI Boundaries

**Ratio this week: 85% Manual / 15% AI**
**Habit introduced: "Compare and contrast."**
**Shift from last week: You get 5% more AI room, and you start formally comparing your work to AI output.**

This week is the first time you are allowed to let AI write a bit of code for you -- but only after you have already written the same thing yourself. The discipline is comparison, not generation. CSS is the ideal week for this because the results are visual: you can see which version looks better and argue about why.

The ratio bumps slightly (from 90/10 to 85/15). The habit evolves: last week you asked AI to explain things; this week you ask it to generate alternatives and then judge them against your own work.

---

## Why The Ratio Moved, Slightly

CSS looks forgiving. Everything is a few words away -- `color: red`, `padding: 20px`, `display: flex`. You will be tempted to ask AI to "make this look nicer" within the first hour.

Resist for the first three days. CSS is not really about the words. It is about the **cascade** (why did my colour lose to a different colour further up the file?), the **box model** (why is there a gap I did not add?), and the **specificity** rules (why does my class not override the element selector?). These three invisible systems are what CSS actually is. You cannot learn them by reading AI output. You can only learn them by writing a broken rule, staring at it, and figuring out why the browser disagrees with you.

After three days of hand-coding, on Day 4 and Day 5, you have earned the right to let AI generate some CSS alongside your own and compare. That is this week's growth.

---

## What You Will Feel This Week

- You will write CSS that looks wrong and not know why.
- You will "fix" it and make it worse.
- You will rage-quit at least once and come back.
- When you finally get a flexbox nav bar to sit right, you will feel like a wizard.

That cycle is the lesson. CSS is the first subject where the computer fights back visually. Welcome to the fight.

---

## What You MUST Do Manually (85%)

### Day 1 -- CSS fundamentals
- Write every selector type by hand: element, class, id, descendant, child, pseudo-class. Test each one in the browser.
- Colour values in hex, rgb, and hsl. Pick a colour by reading the hex digits, not by asking AI to convert.
- Units: px, em, rem, %, vh, vw. Build one small page using each unit at least once so you know what they feel like.

### Day 2 -- The box model, by hand
- Draw the box model on paper. Margin, border, padding, content. Label each side of one box.
- Write a CSS file with deliberate `margin: 20px` and `padding: 20px` on the same element and use DevTools to confirm the box is behaving the way you expect.
- `box-sizing: border-box` vs `content-box` -- build the same layout both ways and observe the difference.

### Day 3 -- Layout with flexbox
- Every `display: flex` rule typed by you, not generated. Every `justify-content`, `align-items`, `flex-direction`, `flex-wrap` -- by hand.
- Build a horizontal nav, a vertical sidebar, and a card grid -- all with flexbox, all manually.
- When something does not centre, open DevTools, inspect the box, and work out which property is missing. Do not ask AI.

### Day 4 -- CSS Grid and responsive design
- Build one layout with CSS Grid that could have been flexbox. Build another that really needs Grid. Feel the difference.
- Write at least three media queries by hand (`@media (max-width: 768px) { ... }`). Test by resizing the browser.

### Day 5 and weekend -- Polish your Week 1 portfolio
- Add CSS to the multi-page semantic HTML portfolio you built on Week 1 weekend. Make it real.
- Every file you touch must still compile. Commit after each major visual change.

---

## You Must Break Things On Purpose

Keep the Week 1 discipline: break CSS at least three times this week and fix it.

Examples:
- Delete a closing `}` and see what the cascade does to the rest of the file.
- Change `display: flex` to `display: block` on your nav and observe.
- Set `position: absolute` on a parent and see what happens to children.
- Delete your CSS reset and see the browser defaults come back.

The fear of breaking is the same fear as the fear of shipping. Kill both now.

---

## What You CAN Use AI For (15%)

**Two allowed uses this week: explanations (continued from Week 1) and comparisons (new this week).**

### Explanations (same as Week 1)
You can still ask "what does this rule do?" after you tried it. The 10-minute rule applies.

### Comparisons (new this week)

After you have written a CSS section manually and it works, you may ask AI to rewrite the same section and give you a second version. Then compare:

- Which one is shorter?
- Which one is more readable?
- Which one uses newer CSS features (grid, custom properties, clamp, logical properties)?
- Is the AI version actually better, or just different?
- Did the AI hallucinate a property that does not exist?

Write down your answer to each question in your daily reflection. The discipline is not "pick the better one" -- it is "develop an opinion and be able to defend it".

### Good vs bad prompts this week

**Bad:** "Make my nav bar look professional."
**Good:** "Here is my flexbox nav [paste]. It works. Can you show me a version using CSS Grid? I want to compare them."

**Bad:** "Why is my CSS broken?"
**Good:** "I expected the `.card` class to have a 2px blue border but it has no border. Here is the rule [paste]. DevTools shows the rule crossed out with strikethrough. What is making the browser ignore it?"

**Bad:** "Write responsive CSS for this page."
**Good:** "Here are my two media queries [paste]. The layout breaks at 900px wide. Which of these rules is likely the cause?"

The pattern is the same as Week 1: you have tried, you have a specific symptom, you ask one focused question. This week you also earn the right to ask for a second version to compare against -- but only *after* your own version works.

---

## The Compare And Contrast Habit

This is the most important learning tool in the entire programme. Build the muscle now, while stakes are low.

The shape:

1. **You build it first.** Write your version completely. Do not look at AI until it works.
2. **Ask AI for its version.** Short, specific prompt with your code attached.
3. **Put them side by side.** Literally -- open two panes in VS Code.
4. **Spot the differences.** Line by line. Property by property.
5. **Judge.** Write one sentence for each difference: "AI used `gap: 1rem` instead of `margin-right: 1rem`. I prefer gap because..."
6. **Pick the version you want to ship.** It can be yours, theirs, or a mix. Your choice. Defend it in one sentence.

By the end of the week, you will have done this five to ten times. That is how you build judgement -- not by following AI and not by rejecting it, but by reading it critically and deciding for yourself.

---

## The 10-Minute Rule (Still)

The rule from Week 1 stays. Before asking AI anything, spend at least ten minutes of real effort:

- Open DevTools and inspect the element.
- Read the MDN page for the property you are using.
- Change one line at a time and reload.
- Read your CSS out loud.

Ten minutes, then ask. Next week this becomes fifteen.

---

## Things AI Is Bad At This Week

- **The cascade.** AI often gives you rules that would be overridden by something else in your file. It cannot see your full stylesheet and know what loses to what.
- **Specificity math.** `.nav a` is more specific than `a`. AI suggestions often forget this and "fix" things by adding more CSS instead of raising specificity.
- **Browser differences.** AI will happily use `-webkit-` prefixes that you do not need, or skip them on a property that does need them on certain browsers. Test every generated rule in at least two browsers.
- **Your HTML.** AI does not know your HTML structure unless you paste it. It will invent class names that do not exist in your markup. Verify every selector against your actual HTML.

When you catch AI inventing a class, note it in your AI Audit. The fact that you caught it is more valuable than the thing AI was trying to generate.

---

## Core Mental Models For This Week

- **The cascade** = "last one wins, unless the earlier one was more specific or !important"
- **The box model** = "margin, border, padding, content -- every element is four concentric rectangles"
- **Specificity** = "inline > id > class > element. Count them before you argue."
- **Document flow** = "elements stack top to bottom unless you tell them otherwise"

If a layout does not look right, one of these four models is the answer. Never ask AI before checking all four yourself.

---

## This Week's AI Audit Focus

In your `AI_AUDIT.md` this week, include a section called **Comparisons**. For every time you asked AI for an alternative version, write:

```markdown
### [Feature name]
- **My version:** [paste or summary]
- **AI version:** [paste or summary]
- **Key differences:** [2-3 bullets]
- **Which did I pick:** [mine / theirs / hybrid] and why
- **Did AI hallucinate anything?** [yes / no -- if yes, what]
```

Facilitators will read these. The quality of your comparisons is this week's grade, not the prettiness of your CSS.

---

## Assessment

The Week 2 assessment is a design review on your CSS-styled portfolio. You open your site, share your screen, and the facilitator asks:

- Point to a rule in your CSS that you are proud of. Why?
- Point to a rule you are not sure about. Walk me through what it does.
- This nav bar uses `display: flex`. What happens if I change it to `display: block`? (Live change. Do it on the spot.)
- Your page has a `color: red` on a paragraph. Where is the red coming from? Is it on the `<p>` or is it inherited?
- Show me a media query and explain why it fires at that breakpoint.
- Show me one AI Audit comparison. Defend your choice in thirty seconds.

**No AI during the assessment.** Stuck is data. Stuck that you can name is progress.

### Live Rebuild Check

The facilitator may delete one CSS rule from your file and ask you to rewrite it from memory. If you cannot, that is not a failure -- it is a flag. Come back next week and prove you learned it.

---

## One Sentence To Remember

"Judgement is the skill -- comparison is the practice." Build your own opinion about every piece of CSS you ship this week, whether you wrote it or AI did.
