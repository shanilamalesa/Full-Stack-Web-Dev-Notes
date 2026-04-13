# Week 27 - AI Boundaries

**Ratio this week: 55% Manual / 45% AI** (ARCHITECTURE DIP)
**Habit introduced: "Design is human work."**
**Shift from last week: Biggest manual bump of the Capstone Sprint. This is architecture week for the African SME OS.**

Capstone Week 1 is spent almost entirely off keyboard: on paper, whiteboard, and conversation. You design the SME OS architecture, the multi-tenant data model, the API surface, and the repo layout. You write almost no code. When you do write code, it is scaffolding for the project structure, not features.

The manual ratio is the highest since Week 10 -- deliberately. Architecture decisions outlive the code. A bad architecture will cost you three weeks of Capstone you do not have. Taking a full week to get it right is the most valuable thing you can do.

---

## Why The Ratio Jumped

Weeks 16-26 gave you tactics: how to wire a payment, how to queue a job, how to split a service. The Capstone asks a different question: *given all these tactics, what do you build?* That is a strategy question. Strategy is a human skill AI cannot substitute for. You can use AI to critique your strategy, but you cannot use AI to form it.

---

## What You MUST Do Manually (55%)

### Day 1 -- Capstone kickoff
- Read the Capstone Sprint brief end-to-end. Do not let AI summarise it.
- Write your own one-paragraph description of the SME OS in your own words. This is the product brief you will refer back to when AI tempts you off-course.

### Day 2 -- Multi-tenant patterns
- Read three different multi-tenant articles. Take notes on paper. Compare the approaches.
- Decide: database-per-tenant, schema-per-tenant, or row-level security? Write down why.
- For the SME OS, row-level security is almost certainly the right answer -- but you must come to that conclusion yourself.

### Day 3 -- Schema design
- Draw the full data model on paper first. Every table, every relationship.
- Then write the schema as SQL by hand. Include RLS policies where relevant.
- Load it into a fresh Postgres database. Verify it works with two tenants.

### Day 4 -- API design
- Design every public endpoint on paper. Method, path, request shape, response shape, errors.
- Write one one-page API document. This is your contract with your future self.
- Start the Express/Next.js scaffolding. Only after the doc is finished.

### Day 5 -- Repo scaffolding and milestones
- Create the repo, the folders, the initial commit.
- Write a MILESTONES.md: what ships by end of Week 28, 29, 30.
- Push to GitHub. Share with cohort.

---

## What You CAN Use AI For (45%)

- **Critique of your design.** "Here is my schema [paste]. What am I missing for multi-tenancy?"
- **Comparison of alternatives.** "I'm choosing between RLS and schema-per-tenant. What are the operational trade-offs of each for a team of one?"
- **Scaffolding the repo structure** after you decided it.
- **Writing the initial README.**

Forbidden:
- Designing the schema.
- Choosing the multi-tenant pattern.
- Designing the API endpoints.
- Deciding what ships in which week.

### Good vs bad prompts this week

**Bad:** "Design a multi-tenant SaaS schema for me."
**Good:** "Here is my draft schema [paste]. I am using RLS for tenant isolation. The `orders` table has `tenant_id`. I enabled RLS and wrote a policy: `USING (tenant_id = current_setting('app.tenant_id')::uuid)`. What attack scenario would bypass this?"

**Bad:** "What should my API look like?"
**Good:** "Here is my API doc [paste]. I have 12 endpoints across auth, users, billing, and reports. Is there an obvious missing endpoint that a multi-tenant SaaS admin would need?"

---

## The Design-Is-Human-Work Habit

This habit is not new -- it has appeared in Weeks 10, 12, 15, 22, 24. The difference in Week 27 is that it is the *whole week*. You do not code much. You think, draw, write, and critique.

When AI tries to do your thinking, stop. Put the keyboard down. Pick up the pen.

---

## Things AI Is Bad At This Week

- **Your actual business.** AI does not know what an SME in Nairobi pays for, what features they would abandon, what their pain is. Only customer conversations can answer that. AI cannot substitute.
- **Feature scope.** AI will happily add features. Your job is to cut.
- **Operational cost.** "Postgres per tenant" is a valid choice in principle and a disaster for a team of one. AI rarely flags this.

---

## Core Mental Models For This Week

- **The Capstone is a startup, not a homework assignment.** Every decision is a product decision.
- **Multi-tenancy is a deployment pattern.** Pick based on your scale, team, and operational budget.
- **A schema migrated badly is a tax for years.** Design carefully.

---

## This Week's AI Audit Focus

Include: your paper diagram, your API doc, your schema file, and your MILESTONES.md. The audit itself is short this week because you wrote little code; the artefacts are the deliverable.

---

## Assessment

- Present your architecture to the cohort. 10 minutes. No slides; just your paper diagram and your thinking.
- Facilitator asks hard questions: "what happens when tenant A's webhook times out?" "What if a single tenant has 10x the load of the others?" You answer with your design choices.

---

## One Sentence To Remember

"The pen beats the keyboard this week. Every week actually, but especially this one."
