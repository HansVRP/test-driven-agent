---
name: delegated-delivery
description: "Use when a request needs several implementation steps, multi-file mechanical edits, focused testing, or independent review. Coordinates bounded Repository Worker tasks and the Quality Gate."
argument-hint: "Implementation task and acceptance criteria"
---

Lead owns scope and final proof. Workers do bounded execution when delegation is worth the handoff.

## Workflow

1. Choose a risk tier. Trivial/docs-only work stays direct. For every non-trivial task, show a numbered `Plan:` and track its steps in todos. Routine, bounded, reversible changes use a short plan and focused check, not a full delegated pipeline. Use the eight-step workflow only for high-risk changes: security/privacy, data loss/migration, public contracts, broad behavior, or unresolved requirements.
2. Follow the plan; replan only when discovery changes scope, and explain the change. Ask only about consequential unresolved choices; use `ASSUMED:` for low-risk decisions. Use Repo Recon when ownership or structure is genuinely unknown.
3. For high-risk behavior, settle observable acceptance criteria. Write and show `spec.md` only when it resolves a real ambiguity; do not wait for approval when requirements are already clear. Test Author scenarios/tests and user approval are reserved for risks or behavior decisions that benefit from independent confirmation. Confirm failing tests fail for the intended reason; protect them in `protected-tests.md`.
4. Delegate named files, acceptance criteria, protected-test paths, and one focused check to Repository Worker. Worker applies `ponytail`, validates its change, and must not edit protected tests. On `TEST-DISPUTE:`, resolve the issue before proceeding; never weaken a test silently.
5. Use Quality Gate for high-risk changes or when independent review materially reduces risk. Give it scope, risk focus, and validation evidence; do not paste diffs. Pick its model per the lead's model split: never the implementer's or Test Author's model; Sonnet for routine, premium for high-risk. The lead checks cited hunks and adjudicates findings.
6. Run mutation checks only for critical risks where a surviving mutant would be consequential. Allow one repair cycle for BLOCKER/NEEDS CHANGES, then re-review; escalate unresolved findings.

## Guardrails

- Keep one edit slice active at a time.
- Protected tests change only with explicit permission. Quality Gate verifies via diff.
- Worker stops after two setup or recovery attempts, 120 seconds without useful output, or an interactive/credential/service blocker.
- Do not delegate destructive or external actions.
- Do not accept worker summaries without checking the changed files or diff.
- Treat executable checks as primary correctness evidence. The Lead adjudicates review findings but reads only cited hunks or disputed risk surfaces instead of duplicating a clean review.
- Stop at the requested behavior. Do not opportunistically refactor.