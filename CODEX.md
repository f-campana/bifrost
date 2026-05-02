# CODEX.md — System Prompt

Read this file completely before taking any action.

---

## What this system is

Sanctum + Bifrost is a human-orchestrated agent operating system.

- **Sanctum** (`sanctum/`) — the compiled knowledge wiki. Private repo.
- **Bifrost** (`bifrost/`) — the schema and operating system for agents. Public repo.
- **Runestone** (`runestone/`) — the design identity browser. Built in Phase 1.

The human orchestrates. Agents execute. No agent selects the next agent.
No agent modifies another agent's output without being explicitly dispatched.

---

## Canonical vocabulary

Use these terms exactly. Do not substitute synonyms.

| Term | Meaning |
|---|---|
| Rune | A named, versioned design identity in Sanctum |
| Design Contract | A project-specific file instanced from a Rune (`DESIGN_[PROJECT].md`) |
| Runestone | The browsable Rune library interface |
| Promote Gate | Human review step before agent output becomes durable knowledge |
| Inbox | `sanctum/00 Inbox/` — where all agent output lands first |
| Loop-back | Quality gate wrapper that retries agents on failure |
| Domain Deepening | research → model → validate → spec, applied per domain |

---

## Reserved names — never use as system components

- **Codex** — this is the harness name (OpenAI). Not a system component.
- **Forge** — belongs to Ropau's public system.
- All Ropau Rune names: Argent Décisif, Néon Humaniste, Signal Brut,
  Encre Éditoriale, Aube Opérative, Cristal Cartographique, Encre Souveraine,
  Flux Bienveillant, Prisme Analytique, Toile Narrative, Crimson Rain.
  These are their intellectual property. Do not use them as our own.

---

## Source of truth locations

| Document | Canonical path |
|---|---|
| System spec (markdown) | `bifrost/system-spec-v0.3.md` |
| System spec (HTML) | `bifrost/system-spec-v0.3.html` |
| Design pipeline contract | `bifrost/contracts/design-pipeline-contract.md` |
| Press Rune | `sanctum/10 Knowledge/Design/runes/press.md` |
| System glossary | `sanctum/10 Knowledge/Concepts/system-glossary.md` |
| Routing guide | `sanctum/10 Knowledge/Design/routing-guide.md` |
| Design principles | `sanctum/10 Knowledge/Design/design-principles.md` |
| DESIGN_RUNESTONE.md | `runestone/DESIGN_RUNESTONE.md` |

---

## Rules — read before every task

**Copy, never regenerate canonical files.**
If a task involves placing a known file into the system, copy it verbatim.
Do not summarise it, rewrite it, or regenerate it from memory.
After copying, verify by checking line count and grepping for a specific
known string from the file.

**One thing at a time.**
Complete one file or one action fully before moving to the next.
Report the result of each step individually.

**Write to Inbox first.**
All agent output goes to `sanctum/00 Inbox/` before any human review.
Nothing is promoted to `10 Knowledge/` or `30 Memory/` by the agent.
The human promotes.

**Never collapse the repo split.**
Sanctum and Bifrost are separate git repos. Never suggest merging them.
Runestone is a separate directory, not a subdirectory of either repo.

**Runestone uses pnpm.**
For Runestone, use pnpm, not npm. The `packageManager` field in
`runestone/package.json` is the source of truth for the exact pnpm version.

**Component primitives: Base UI.**
The Press component library uses @base-ui-components/react
as the behavioral primitive layer for all interactive
components. Radix UI primitives and react-aria hooks are
not the default. asChild is not used. The full architecture
is documented in:
sanctum/10 Knowledge/Concepts/
press-component-architecture-adr.md
Read that document before producing or reviewing any
React component.

**Ralph loop goals: three required elements.**
A /goal spec must include all three of the following
or the loop will fill gaps with its own judgment:

1. Success criteria — binary, machine-verifiable
   conditions that prove the goal is complete.
   Example: "pnpm exec tsc --noEmit exits zero"
   not "the component looks correct".

2. BLOCKED conditions — explicit situations where
   the loop must stop and report rather than solve.
   Always include:
   - Achieving any gate requires patching node_modules
     or any installed dependency
   - Achieving any gate requires modifying files not
     created or explicitly listed in this goal
   - Pre-existing errors in unrelated files do not
     block this goal — exclude them explicitly if
     needed (e.g. exclude scripts/ from tsconfig)

3. Context — what the loop should assume rather than
   discover. Include: correct package names and
   versions, any known gaps in the ecosystem
   (e.g. "Base UI has no Anchor primitive at v1.4.1,
   use native &lt;a&gt;"), and which files require
   modification.

Evidence: two PressButtonLink ralph loop runs.
Run 1 — wrong package name + no BLOCKED conditions
→ loop patched node_modules to satisfy gates.
Run 2 — correct context + explicit BLOCKED conditions
→ 1 loop, 257 seconds, clean output, no side effects.
Full records in:
sanctum/40 Sources/Design/sessions/
pressbuttonlink-v2-loop-2026-05-01.md

**Verification is mandatory.**
After every file write, verify the file exists at the correct path.
After every copy, verify the content matches the source by line count
and spot-check.

**Never invent vocabulary.**
If a term is not in the canonical vocabulary above or in the system
glossary, do not introduce it. Surface the gap instead.

**99 Templates/ is the format reference.**
Every new skill uses `sanctum/99 Templates/SKILL.md` as its template.
Every new Rune uses `sanctum/99 Templates/rune.md`.
Every new session summary uses `sanctum/99 Templates/session-summary.md`.

**Route restructuring must enumerate removals in the
same commit as the move.**
When a dispatch moves content from one route to
another, list:
- every component or section that moves
- every component or section that must be removed
  from the origin route
- every CSS block, helper, or style selector that
  becomes dead after the move

All removals — including dead style selectors — ship
in the same commit as the move. Do not report a route
split complete until the destination route renders the
moved content, the origin route no longer renders it,
and no dead styles for it remain in the tree.

**Agent self-edits may not silently remove previously
instructed code.**
If you remove any code that was added earlier in the
same dispatch chain, call it out explicitly before
editing and again in the final report.
Required report line:
"Self-edit removal: [file] — removed [what] —
reason [why]."
If the removal changes behaviour on any viewport or
input mode, name that behaviour explicitly.

**Component moves require copy review in the same
dispatch.**
When a component moves between routes, review:
- intro copy
- captions
- call-to-action labels
- instructional notes

Do not preserve route-specific framing by default.
Re-check whether the moved copy still matches the new
page purpose, audience, and interaction model.

**Every dispatch that produces commits must write an
Inbox run report.**
The run report is the agent's account of what
happened — distinct from commit messages, which
describe intent, and from the dispatch brief, which
describes intent before the fact.

The report lives at sanctum/00 Inbox/[skill-or-task]-
[target]-[date].md and contains:
- dispatch summary (what was asked)
- actions taken (file by file)
- commands run with verbatim output for any
  verification step the dispatch named
- deviations from the brief, if any
- commit SHAs produced

Commit messages may be cited as evidence of what
changed. They may not stand in for the run report.

---

## Context boot for every session

Read these files in this order before beginning any implementation work:

1. This file (`bifrost/CODEX.md`)
2. `bifrost/system-spec-v0.3.md`
3. `bifrost/AGENTS.md`
4. `sanctum/AGENTS.md`
5. The relevant skill or contract for the current task

Do not begin work until you have read all five.

---

## Current phase

**Phase 0 — Foundation.**

What is done:
- Repos scaffolded
- Canonical files committed to both repos
- Press Rune written
- Design pipeline contract written
- Five design SKILL.md files written
- DESIGN_RUNESTONE.md instanced from Press
- Session summary ingested into 40 Sources/

What remains in Phase 0:
- 99 Templates/ populated (all template files)
- GitHub read skills written (github/read-file)
- Manual loop closed once end-to-end: dispatch one skill, output lands
  in Inbox, human promotes — before any automation is added

Do not proceed to Phase 1 work (Runestone build, dialogue skill,
harvester skill) until the manual loop is proven.

---

## When in doubt

Stop. Write your uncertainty to `sanctum/00 Inbox/harness-feedback.md`.
Do not guess. Do not expand scope. Surface the question and wait.
