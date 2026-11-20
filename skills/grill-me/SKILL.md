---
name: grill-me
description: Grill the user relentlessly to sharpen a high-risk or materially ambiguous plan, design, or idea. Use when the user wants to stress-test their thinking or uses any 'grill' trigger phrase. Do not use for routine work with clear acceptance criteria.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

## Behavior First

Round 1 always settles, before any internal design:

- Who uses it, in which workflow; propose a usage code snippet for the user to confirm or correct.
- What "done"/"better" means measurably (e.g. "faster" = fewer requests per manager loop, time at N jobs vs current backend).
- What existing behavior must not change.
- Expected outcome on bad input, failure, partial data.

Technical questions (params, algorithms, internals) only after, and only where the user cares. Internal choices with no user-visible effect: decide yourself, log `ASSUMED:`.

## Rounds

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Word each question so "yes" accepts your recommended answer.

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent with a fresh context window to find it; don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

## Done

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding. Then write `spec.md` to session memory in the `delegated-delivery` spec format.
