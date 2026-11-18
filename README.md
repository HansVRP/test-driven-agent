# Test Driven Agent

Risk-based development plugin for VS Code Copilot. Routine tasks take a direct, focused path; high-risk work can use spec-first testing, protected tests, implementation, and independent review. The lead owns intent, decisions, and verification; agents handle bounded work when isolation or parallelism is worth the handoff.

## Agents

| Agent | Models | Tools | Role |
| --- | --- | --- | --- |
| Test Driven Agent (lead) | Claude Opus 5.5 -> GPT-6.1 Sol | Ask, memory, skills, read, edit, delegate | Plans with the user, delegates, decides, checks |
| Research Scout | GPT-6.1 Sol -> Claude Sonnet 5.5 | All (read-only by instruction) | Web research via built-in search and fetch, repository maps |
| Repository Worker | Claude Sonnet 5.5 -> GPT-5.3-Codex | All | Named-file edits, builds, tests, scripts |
| Quality Gate | GPT-6.1 Sol -> Claude Opus 5.5 | Read, search, shell, skills | Independent correctness, security, minimality review |
| Test Author | GPT-6.1 Sol -> Claude Opus 5.5 | Read, search, edit, shell, skills | Writes failing behavior tests from spec + current code/tests, before implementation |
| Quick Helper | Claude Haiku 5.5 | Read, search, edit, shell | Fully specified repetitive work |

Scout and Worker get every tool, so MCP servers you add are available to them in new sessions.

## How It Works

- **Risk tiers:** trivial/docs-only work stays direct. Routine, bounded changes get one focused check. High-risk changes (security/privacy, data loss/migration, public contracts, broad behavior, or unresolved requirements) can use the full spec-first workflow. No automatic approval gates when requirements are clear.
- **User involvement:** ask about consequential unresolved choices, not internal implementation details. Subagents return `QUESTION:` instead of guessing and `ASSUMED:` for low-risk choices. Record decisions when they affect later work or handoffs.
- **Delegation:** delegate external facts, broad discovery, noisy output, multi-file edits, and independent review when isolation or parallelism is worth its overhead. Give named scope and acceptance criteria; subagents return capped summaries, never transcripts.
- **Review loop:** explicit review requests and high-risk changes go to Quality Gate; use Research Scout first for unknown repos. Split large reviews only across independent areas. Worker validates delegated changes; allow one focused repair and re-review, then escalate unresolved findings.
- **High-risk TDD:** when independent assurance is useful, the Test Author derives scenarios and failing behavior tests from the settled spec, without seeing implementation changes. Tests are protected from implementation edits. Mutation checks are reserved for critical risks where a survivor would matter. See `behavior-tests` and `delegated-delivery` for detailed criteria.
- **Plans and progress:** every non-trivial task starts with a numbered plan and matching todos. Routine work uses a short plan; high-risk work uses the eight-step plan. Follow the plan and explain any replan; avoid repetitive status narration.
- **Lean output:** instructions and answers are written caveman-style; no other progress narration or check-ins, final answer ≤8 lines. Negations, conditions, and uncertainty are kept.
- **Cost control:** use the least costly available model and context that reliably handles the task; escalate for difficult reasoning, high risk, or a failed first pass. Compare a small set of representative tasks for quality, rework, and cost before changing defaults.
- **Ponytail:** one minimal-change guide for implementation and review.
- **Output trimming:** a `postToolUse` hook ([scripts/trim-output.js](scripts/trim-output.js)) cuts shell, search, and fetch output over 8000 characters to head, tail, and error lines, and saves the full output to a temp file.

## Skills

| Command | Purpose |
| --- | --- |
| `/delegated-research` | Source-backed research through Research Scout |
| `/repo-recon` | Repository map without filling lead context |
| `/delegated-delivery` | Bounded worker execution plus independent review |
| `/behavior-tests` | Risk-first behavior tests: Given/When/Then, public API, fakes at boundaries, mutation check. Always loaded by Test Author; general rules adapted from [mavka-ai/unit-tests-skills](https://github.com/mavka-ai/unit-tests-skills) and [obra/superpowers](https://github.com/obra/superpowers) `writing-good-tests` (both MIT) |
| `/grill-me` | Relentless spec interview (wraps `grilling`, from [mattpocock/skills](https://github.com/mattpocock/skills)) |
| `/ponytail` | Lazy-senior minimal-change mode (vendored from [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)) |

## Origin

This plugin started from Victor Verhaert's personal agent setup and was adapted into a risk-based, test-driven workflow.
https://github.com/VictorVerhaert/personal-assistant

## Install

Load the folder as a local agent plugin or add it to a Copilot plugin marketplace, then select **Test Driven Agent**. Requires Node.js for the hook.

Built for the Copilot harness; the Local harness still works, without output replacement from the hook or `reasoning-effort`. Tool lists include both harness names; unknown names are ignored, so editor "unknown tool" warnings for `ask_user`, `store_memory`, or `skill` are expected before VS Code 1.144.

Web research uses the harness's built-in web search and fetch tools; no MCP server or API key is needed.

**Model/context:** use the smallest available context that fits the task; increase it only when the work needs more repository or conversation history. The lead model list puts Claude Opus 5.5 first, with GPT-6.1 Sol as the fallback.