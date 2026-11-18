---
name: behavior-tests
description: "Use when planning, writing, or reviewing tests. Risk-first, behavior-driven tests through the public API, fakes at external boundaries, tests as usage docs, mutation check proving tests catch bugs. Language-agnostic."
---

Tests prove OUR behavior and decisions to a user. Not coverage, not library behavior, not how the code works inside.

## 1. Plan: risks first

Before any scenario, read spec, current code, existing tests/examples of the area. List what could go wrong in OUR logic, one line each (what breaks, who notices, how bad):

- silent wrong results, data loss/corruption, security, regressions of existing behavior, boundaries (0, 1, many, page edges), failure/partial responses, compatibility.
- Not library/framework behavior (pandas, pystac, bcrypt, datetime math).

Then scenarios from risks + acceptance criteria, Given/When/Then, each linked to a risk. Mostly edge, boundary, error paths; few happy paths. One end-to-end scenario of the main user workflow (real orchestrator/loop over the new component, external services faked).

## 2. Write

- One test = one scenario. Name reads as sentence: `test_<who/what>_<does>_<when>` (`test_manager_finishes_all_jobs_with_stac_db`).
- Body Arrange-Act-Assert with `# Given / # When / # Then`. Setup inside the test (cause next to effect), not hidden in distant fixtures; shared fixtures only for boilerplate.
- Act through public API as a user writes it. Tests double as usage docs.
- Assert observable outcome: returned data, persisted state, raised error, end-to-end result, round trip. `actual`/`expected` names.
- Literal expected values, derived by hand, never computed with the code under test or its helpers (mirror assertion = always true). No logic in tests (no loops/conditionals). KISS over DRY.
- No change detectors: not `assert page_size == 1000`, but the behavior that depends on it ("1000 jobs come back in one request").
- Explicit test data; never rely on hidden defaults for values the assertion depends on.
- Real objects and fakes (tmp dirs, in-memory/fake server, `requests_mock`, project test doubles) over mocks. Fake only external boundaries (network, clock, remote service). Before faking, list the real method's side effects; keep the ones the test depends on real.
- Fake responses mirror the complete real structure (all documented fields), not only the fields the code reads today.
- A mock earns no assertions: asserting on the mock itself tests the mock. Unmock or delete.
- Mock setup over half the test: switch to real components.
- Forbidden unless spec names it as user contract: asserting mock call args/counts of internals, private attributes, default constants, exact query strings, which internal method was used.
- Performance requirement: assert cost at the faked boundary scaling with data (requests or items fetched for N), or compare against old implementation on same data. No wall-clock thresholds.
- Match style of 1-2 sibling test files; reuse their fixtures.
- Kept behavior ("must not change"): regression tests, reuse existing ones that already cover it.

## 3. Self-check each test

1. Name the production change that would make it fail. None, only an intentional decision (change detector), or "library works": drop or retarget to behavior.
2. Breaks if the implementation is rewritten with same behavior? Yes: rewrite or drop.
3. Would a newcomer learn usage from it? No: rename/restructure.

## 4. Prove: red, then mutation

- Before implementation: run, confirm fail for the right reason (missing behavior), not import/syntax/fixture errors. New test passes immediately: it tests existing behavior; fix the test.
- Green means the project's full suite, not only the new file.
- After green: record `git diff` of production files. Per risk, apply one small mutation that reintroduces it (off-by-one, dropped filter, skipped page, swallowed error, wrong default, missing side effect, empty return, missing validation), run tests, restore exactly. Confirm production diff equals recorded one. Report killed/survived; survivor = missing test. If a mutation tool is installed (`mutmut`, PIT, Stryker), may run it scoped to changed files instead.

## Review use

Reviewing tests: flag internal-coupled assertions, change detectors, mirror assertions, assertions on mocks, partial fake data, library tests, logic in tests, missing risk coverage, missing end-to-end test.
