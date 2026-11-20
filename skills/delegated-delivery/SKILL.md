---
name: delegated-delivery
description: "Use when a request needs several implementation steps, multi-file mechanical edits, focused testing, or independent review. Coordinates bounded Repository Worker tasks and the Quality Gate."
argument-hint: "Implementation task and acceptance criteria"
---

Canonical workflow. Lead owns scope and final proof. Workers do bounded execution when delegation is worth the handoff.

## Risk Tiers

- **Trivial:** explanation, docs-only, or mechanical change with no behavior impact. Act directly; no plan or delegation.
- **Routine:** bounded, reversible, low-risk behavior change. Short plan: (1) inspect scope and choose a focused check, (2) make the smallest change, (3) run the check and report. No spec, Test Author, mutation pass, or separate review required. No approval pause when request and acceptance criteria are clear.
- **High-risk:** security/privacy, data loss or migration, public contract changes, broad/cross-cutting behavior, or unclear acceptance criteria. Use the eight-step plan. Apply safeguards proportionally; skip handoffs that do not reduce risk.

For every non-trivial task, show a numbered `Plan:` and track the same steps in todos. Replan only when discovery changes scope, and explain why. Mark skipped conditional steps with a brief reason. Ask only about consequential unresolved choices; use `ASSUMED:` for low-risk decisions. Use Repo Recon when ownership or structure is genuinely unknown.

## High-Risk Plan

1. Clarify behavior, acceptance criteria, and risks; ask only unresolved consequential questions (`grill-me` if materially ambiguous).
2. Write a short `spec.md` when it clarifies the contract; settle remaining decisions; get user sign-off before step 6.
3. Identify risk-based usage scenarios.
4. Write behavior tests independently (Test Author) when that adds assurance.
5. Confirm tests fail for the intended reason; protect them in `protected-tests.md`.
6. Implement the smallest change without editing protected tests.
7. Run focused/full validation; mutation checks only for critical risks where a surviving mutant would be consequential.
8. Review independently (Quality Gate) and report evidence.

`spec.md` (short): goal + who uses it; usage example as the user would write it; observable acceptance criteria (returned data, persisted state, errors, end-to-end result, measurable cost; not internal params unless the user named them as contract); must-not-change; non-goals; constraints.

## Handoffs

- **Repository Worker:** named files, acceptance criteria, protected-test paths, one focused check. Worker applies `ponytail`, validates its change, and must not edit protected tests. On `TEST-DISPUTE:`, resolve before proceeding; never weaken a test silently.
- **Quality Gate:** for high-risk changes or explicit review requests. Give scope, risk focus, and validation evidence; no pasted diffs. Model: never the implementer's or Test Author's; Sonnet for routine, Opus (or Sol if Opus was used) for high-risk.
- **Findings:** lead checks cited hunks and adjudicates. Allow one repair cycle for BLOCKER/NEEDS CHANGES, then re-review; escalate unresolved findings to the user with evidence.

## Guardrails

- Keep one edit slice active at a time.
- Protected tests change only with explicit permission. Quality Gate verifies via diff.
- Worker stops after two setup or recovery attempts, 120 seconds without useful output, or an interactive/credential/service blocker.
- Do not delegate destructive or external actions.
- Do not accept worker summaries without checking the changed files or diff.
- Treat executable checks as primary correctness evidence. Read only cited hunks or disputed risk surfaces instead of duplicating a clean review.
- Stop at the requested behavior. Do not opportunistically refactor.
