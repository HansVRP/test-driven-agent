---
name: grill-me
description: A thorough interview to sharpen high-risk or materially ambiguous plans and designs. Do not use for routine work with clear acceptance criteria.
---

Load the `grilling` skill and follow it.

Behavior first. Round 1 always settles, before any internal design:

- Who uses it, in which workflow; propose a usage code snippet for the user to confirm or correct.
- What "done"/"better" means measurably (e.g. "faster" = fewer requests per manager loop, time at N jobs vs current backend).
- What existing behavior must not change.
- Expected outcome on bad input, failure, partial data.

Technical questions (params, algorithms, internals) only after, and only where the user cares. Internal choices with no user-visible effect: decide yourself, log `ASSUMED:`.

When the frontier is empty and the user confirms, write `spec.md` to session memory in the lead's spec format (goal, usage example, observable acceptance criteria, must-not-change, non-goals, constraints).
