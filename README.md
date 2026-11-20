# Test Driven Agent

Risk-based development plugin for VS Code Copilot. Routine tasks take a direct, focused path; high-risk work can use spec-first testing, protected tests, implementation, and independent review. The lead owns intent, decisions, and verification; agents handle bounded work when isolation or parallelism is worth the handoff.

## Agents

| Agent | Models | Tools | Role |
| --- | --- | --- | --- |
| Test Driven Agent (lead) | Claude Opus 5.5 | Ask, memory, skills, read, edit, delegate | Plans with the user, delegates, decides, checks |
| Research Scout | Claude Sonnet 5.5 (escalation: Claude Opus 5.5) | Read, search, web, skills (read-only) | Web research via built-in search and fetch, repository maps |
| Repository Worker | GPT only: GPT-5.3-Codex (escalation: GPT-6.1 Sol) | All | Named-file edits, builds, tests, scripts |
| Quality Gate | Per task: Claude Sonnet 5.5 (routine), Claude Opus 5.5 or GPT-6.1 Sol (high-risk); never the implementer's or Test Author's model | Read, search, shell, skills | Independent correctness, security, minimality review |
| Test Author | Claude only: Claude Sonnet 5.5 (escalation: Claude Opus 5.5) | Read, search, edit, shell, skills | Writes failing behavior tests from spec + current code/tests, before implementation |

Worker gets every tool, so MCP servers you add are available to it in new sessions. Scout is limited to read-only tools.

## How It Works

The full workflow lives in one place: [skills/delegated-delivery/SKILL.md](skills/delegated-delivery/SKILL.md). Summary:

- **Risk tiers:** trivial/docs-only work stays direct. Routine, bounded changes get a short plan and one focused check, no approval pause. High-risk changes (security/privacy, data loss/migration, public contracts, broad behavior, unresolved requirements) use an eight-step spec-first plan with user sign-off before implementation, optional independent tests from Test Author (protected from implementation edits), and Quality Gate review.
- **User involvement:** ask about consequential unresolved choices, not internal implementation details. Subagents return `QUESTION:` instead of guessing and `ASSUMED:` for low-risk choices.
- **Delegation:** only when isolation or parallelism is worth the handoff. Subagents return capped summaries, never transcripts.
- **Model separation:** no model both implements and judges. Test Author is Claude-only, Repository Worker GPT-only; Quality Gate never uses the implementer's or Test Author's model. Default to cheaper models; escalate (Sonnet->Opus, Codex->Sol) on repeated failure, unclear root cause, high-risk scope, or conflicting evidence.
- **Lean output:** instructions and answers are written caveman-style; no other progress narration or check-ins, final answer ≤8 lines. Negations, conditions, and uncertainty are kept. Security warnings, irreversible actions, and confused users get full sentences; code, commands, paths, and error messages stay verbatim.
- **Ponytail:** one minimal-change guide for implementation and review.
- **Output trimming:** a `postToolUse` hook ([scripts/trim-output.js](scripts/trim-output.js)) cuts shell, search, and fetch output over 8000 characters to head, tail, and error lines, and saves the full output to a temp file.

## Skills

| Command | Purpose |
| --- | --- |
| `/delegated-research` | Source-backed research through Research Scout |
| `/repo-recon` | Repository map without filling lead context |
| `/delegated-delivery` | Bounded worker execution plus independent review |
| `/behavior-tests` | Risk-first behavior tests: Given/When/Then, public API, fakes at boundaries, mutation check. Always loaded by Test Author; general rules adapted from [mavka-ai/unit-tests-skills](https://github.com/mavka-ai/unit-tests-skills) and [obra/superpowers](https://github.com/obra/superpowers) `writing-good-tests` (both MIT) |
| `/grill-me` | Relentless spec interview for high-risk or ambiguous plans (adapted from `grilling` in [mattpocock/skills](https://github.com/mattpocock/skills)) |
| `/ponytail` | Lazy-senior minimal-change mode (vendored from [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)) |

## Origin

This plugin started from Victor Verhaert's personal agent setup and was adapted into a risk-based, test-driven workflow.
https://github.com/VictorVerhaert/personal-assistant

## Install

Load the folder as a local agent plugin or add it to a Copilot plugin marketplace, then select **Test Driven Agent**. Requires Node.js 18+ on `PATH` for the hook ([hooks.json](hooks.json)); without it the hook fails open and output is untrimmed.

Built for the Copilot harness; the Local harness still works, without output replacement from the hook or `reasoning-effort` (per-agent frontmatter setting for model thinking depth: low/medium/high). Tool lists include both harness names; unknown names are ignored, so editor "unknown tool" warnings for `ask_user`, `store_memory`, or `skill` are expected before VS Code 1.144.

Web research uses the harness's built-in web search and fetch tools; no MCP server or API key is needed.

**Model/context:** use the smallest available context that fits the task; increase it only when the work needs more repository or conversation history. Assumes access to both Claude and GPT models; there are no cross-vendor fallbacks.

## License

MIT, see [LICENSE](LICENSE). Vendored skills keep their original licenses.