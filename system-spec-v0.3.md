# Sanctum + Bifrost — System Specification

**Version:** 0.3  
**Status:** Design pipeline validated. Phase 0 ready to build.  
**Sanctum repo:** private  
**Bifrost repo:** public  
**Last updated:** 2026-04-22

---

## Table of Contents

1. [Overview](#1-overview)
2. [Core Principles](#2-core-principles)
3. [Build Status](#3-build-status)
4. [Sanctum — The Compiled Wiki](#4-sanctum--the-compiled-wiki)
5. [Bifrost — The Schema Layer](#5-bifrost--the-schema-layer)
6. [The Domain-Deepening Pattern](#6-the-domain-deepening-pattern)
7. [Design Pipeline](#7-design-pipeline)
8. [Agent Taxonomy](#8-agent-taxonomy)
9. [Hardware Topology](#9-hardware-topology)
10. [Model Routing](#10-model-routing)
11. [Interface Layer](#11-interface-layer)
12. [AGENTS.md Contract](#12-agentsmd-contract)
13. [SKILL.md Contract](#13-skillmd-contract)
14. [GitHub Skills](#14-github-skills)
15. [Quality Gate](#15-quality-gate)
16. [Loop-back Gate](#16-loop-back-gate)
17. [Promote Gate + Self-Improvement Loop](#17-promote-gate--self-improvement-loop)
18. [The Core Operating Loop](#18-the-core-operating-loop)
19. [Phased Build Sequence](#19-phased-build-sequence)
20. [Conventions](#20-conventions)

---

## 1. Overview

This document specifies a hybrid human-orchestrated agent operating system built from a clean slate. It has two shared infrastructure layers — Sanctum and Bifrost — and any number of product consumers.

**Sanctum** is the compiled knowledge wiki. It owns everything the system knows — sources, synthesis, memory, indexes, project context, and the design identity library. It is maintained by agents but read by both humans and agents. Knowledge compounds here over time.

**Bifrost** is the schema and operating system that governs how agents behave. It owns skills, contracts, routing rules, verification logic, and prompt assembly rules. It tells agents how to operate across all technical domains. Projects consume both.

**Runestone** is the browsable web interface to Sanctum's design identity library. It renders each Rune as a live experience and exposes the routing guide. Built using the design pipeline itself.

The orchestrator is a human. Automation is earned, not assumed. Intelligence lives at the top.

### What this system is not

Not an automated orchestration framework. No AI orchestrator. No agents spawning agents without human dispatch. The system is a set of well-defined scoped agents that a human routes between. Every agent has a narrow scope, a defined input, and a defined output.

### The Karpathy model

This system is a direct implementation of Karpathy's LLM Wiki pattern, generalised into a full agent operating system:

- **Raw sources** (`Sanctum/40 Sources/`) — immutable inputs the LLM reads but never modifies
- **Compiled wiki** (`Sanctum/10 Knowledge/`, `01 Indexes/`, `30 Memory/`) — LLM-maintained synthesis that compounds over time
- **Schema** (Bifrost) — the operating instructions and skills that govern agent behaviour

The key property: the wiki is a persistent compounding artifact. Cross-references are already there. Contradictions have been flagged. Synthesis reflects everything ingested. Good answers become wiki pages. Nothing useful disappears into chat history.

---

## 2. Core Principles

### Human orchestrator
The human makes all routing decisions, task decompositions, and promote gate judgments. No agent decides which agent runs next. Automation is for execution, not orchestration.

### Sources are immutable
Raw source material in `40 Sources/` is never modified by agents. The LLM reads from it but writes only to the compiled zone. Provenance must be traceable.

### The wiki compounds
Every useful synthesis — an answer, an analysis, a connection discovered — gets filed back into the wiki. Nothing valuable disappears into chat history. The system gets smarter with every interaction.

### Bifrost is the schema, not the knowledge
Bifrost defines how the wiki is maintained and how agents behave. Sanctum contains what the wiki knows. These are different concerns and must never blur.

### Harness agnosticism
`AGENTS.md` and `SKILL.md` are the harness contract. Claude Code and Codex both read them. The system does not depend on any single harness.

### Pull before push
Every automated trigger starts as manual dispatch. A role earns automated triggering only after ≥2 weeks of predictable manual use. Automation is a reward for proven reliability.

### Dogfood the build
Each phase uses what was built in the previous phase. The system builds itself. The design pipeline builds the Runestone. The wiki/ingest skill files the notes from its own creation session.

### Full observability
Every gate check is a command you can run yourself. Every agent output is a file you can read. Every failure is structured output you can inspect.

### Promote gate
Nothing moves from agent output to durable wiki without human review. Background agents write to `00 Inbox/` only. You review and approve before anything reaches `10 Knowledge/` or `30 Memory/`.

### Narrow context
Action agents operate on ≤10 files. Workflow agents load one domain map. Project agents load cross-domain maps. Strategy agents (you) hold full context. More context than needed causes scope creep.

### Domain deepening
Each technical domain starts with a lightweight skill and earns a full contract through real use. When agent failures in a domain become specific, recurring, and consequential, invest in the deep process: research → model → validate → spec. The design pipeline is the first completed instance of this pattern. Other domains follow when they qualify.

---

## 3. Build Status

| Capability | Status | Phase |
|---|---|---|
| Sanctum repo | ❌ not yet | Phase 0 |
| Bifrost repo | ❌ not yet | Phase 0 |
| Sanctum folder structure | ❌ not yet | Phase 0 |
| Bifrost folder structure | ❌ not yet | Phase 0 |
| `log.md` in Sanctum | ❌ not yet | Phase 0 |
| Ollama on Mac mini | ❌ not yet | Phase 0 prerequisite |
| Pi on Mac mini | ❌ not yet | Phase 0 prerequisite |
| Symlink `~/.agents/skills/bifrost/` | ❌ not yet | Phase 0 |
| Design pipeline contract | ✅ written | `Bifrost/contracts/design-pipeline-contract.md` |
| Four core Runes | ❌ not yet | Phase 0 — design domain |
| Routing guide | ❌ not yet | Phase 0 — design domain |
| `design/curator` SKILL.md | ❌ not yet | Phase 0 — design domain |
| `design/builder` SKILL.md | ❌ not yet | Phase 0 — design domain |
| `design/validator` SKILL.md | ❌ not yet | Phase 0 — design domain |
| Runestone v1 | ❌ not yet | Phase 1 |
| `design/dialogue` SKILL.md | ❌ not yet | Phase 1 |
| `design/harvester` SKILL.md | ❌ not yet | Phase 1 |
| GitHub read skills | ❌ not yet | Phase 1 |
| Loop-back wrapper | ❌ not yet | Phase 2 |
| `docs/changelog` skill | ❌ not yet | Phase 2 |
| `wiki/ingest` skill | ❌ not yet | Phase 3 |
| `wiki/query` skill | ❌ not yet | Phase 3 |
| `wiki/lint` skill | ❌ not yet | Phase 3 |
| GitHub write skills | ❌ not yet | Phase 4 |
| Slack gateway | ❌ not yet | Phase 5 |
| Tachyon configured | ❌ not yet | Phase 5 |
| `design/distill` SKILL.md | ❌ not yet | Phase 6 |
| Session-promoter agent | ❌ not yet | Phase 6 |

---

## 4. Sanctum — The Compiled Wiki

Sanctum is a git-backed private repository. It is the system's long-term memory, research archive, knowledge base, and design identity library.

### Two zones

**Immutable zone — `40 Sources/`**
Raw inputs. The LLM reads but never modifies. Everything here retains provenance. If a source is wrong, the error lives here permanently and the correction lives in the compiled zone.

**Compiled zone — everything else**
The LLM owns this zone. It creates pages, updates them when new sources arrive, maintains cross-references, flags contradictions, and keeps everything consistent. Humans read it; agents write it (subject to the promote gate).

### Folder structure

```
sanctum/
  README.md
  AGENTS.md
  system-spec.md           ← symlink to bifrost/system-spec.md

  00 Inbox/
    harness-feedback.md    ← agents append AGENTS.md improvement suggestions

  01 Indexes/
    index.md               ← master navigation entry point
    log.md                 ← append-only chronological record of all operations
    knowledge-map.md
    memory-map.md
    projects-map.md
    sources-map.md

  10 Knowledge/
    Articles/
    Concepts/
    Lessons/
    Exercises/
    Design/
      runes/               ← design identity library (Rune files)
        argent-decisif.md
        neon-humaniste.md
        signal-brut.md
        encre-editoriale.md
      routing-guide.md     ← six-question Rune selection guide
      design-principles.md ← cross-Rune motion grammar, axis definitions

  20 Projects/
    _project-template.md
    project-index.md

  30 Memory/
    Environment/
    Preferences/
    Project Memory/

  40 Sources/              ← IMMUTABLE — LLM reads, never writes
    Design/
      sessions/            ← design session summaries for ingestion
      references/          ← external references (Google spec, animations.dev, etc.)
    Notes Imports/
    Web Clips/
    Repos/
    Papers/
    Transcripts/

  60 Assets/
    Images/
    Diagrams/

  90 Archive/

  99 Templates/
    concept-note.md
    lesson-note.md
    memory-note.md
    project-note.md
    source-note.md
    session-summary.md
    research-note.md
    rune.md                ← template for new Rune files
```

`50 Outputs/` is intentionally absent. Transient outputs land in `00 Inbox/` and are either promoted to `10 Knowledge/` or discarded. Per Karpathy: good answers become wiki pages.

### `log.md` — first-class maintenance file

An append-only chronological record of system operations. Every ingest, query, lint pass, and design run gets an entry. Format:

```
## [2026-04-22] design/curator | argent-decisif@1.0.0 → DESIGN_TERRITOIRE.md
## [2026-04-22] wiki/ingest | design-pipeline-session-2026-04-21.md
## [2026-04-22] design/builder | DESIGN_SIGNAL.md → landing-signal.html · run-report filed
```

---

## 5. Bifrost — The Schema Layer

Bifrost is a git-backed public repository. It is the operating system for Sanctum — it defines how agents behave, not what they know.

### Folder structure

```
bifrost/
  README.md
  AGENTS.md
  system-spec.md           ← the document you are reading

  contracts/
    skill-contract.md
    memory-contract.md
    prompt-contract.md
    verification-contract.md
    project-boundaries.md
    design-pipeline-contract.md   ← Phase 0 complete

  skills/
    _index.md
    _template/
      SKILL.md
    design/
      design-curator/SKILL.md
      design-builder/SKILL.md
      design-validator/SKILL.md
      design-dialogue/SKILL.md
      design-harvester/SKILL.md
      design-distill/SKILL.md
    wiki/
      wiki-ingest/SKILL.md
      wiki-query/SKILL.md
      wiki-lint/SKILL.md
    docs/
      docs-changelog/SKILL.md
      docs-drift/SKILL.md
    react/
      react-implement/SKILL.md
      react-test/SKILL.md
      react-review/SKILL.md
    typescript/
      ts-type/SKILL.md
      ts-audit/SKILL.md
    express/
      express-route/SKILL.md
      express-schema/SKILL.md
    github/
      github-read-file/SKILL.md
      github-read-diff/SKILL.md
      github-create-branch/SKILL.md
      github-open-pr/SKILL.md
    memory/
      memory-write/SKILL.md
      memory-summarize/SKILL.md
    architecture/
      arch-solid/SKILL.md
      arch-ddd-review/SKILL.md
    security/
      sec-audit/SKILL.md
      sec-auth-review/SKILL.md

  scripts/
    loop-back.sh
    quality-gate.sh
    skill-lint.sh

  examples/
```

---

## 6. The Domain-Deepening Pattern

The system improves domain by domain. Each domain starts lightweight and earns depth through real use. This is the core mechanism by which the system compounds in quality over time.

### The four stages

**Stage 1 — Research**
Find who has already solved this domain rigorously. Study their approach, vocabulary, and failure modes before writing anything. The design domain referenced Ropau's Design Forge, Emil Kowalski's animation skill system, and Google's design.md specification. External research is not overhead — it compresses months of trial and error into a structured starting point.

**Stage 2 — Model**
Identify the conceptual vocabulary for this domain. Name things precisely. The design domain produced: Rune, Runestone, Design Contract, Axes, Moments, Routing Guide, Run Report. Names that are yours, not borrowed, that fit with the broader system vocabulary (Norse/Marvel register for infrastructure names; precise English for domain concepts).

**Stage 3 — Validate**
Run real work through lightweight specs. Observe what breaks. The design domain ran three complete packages through cold Claude instances — no prior context, only the package files. Three different visual identities. Each failure was specific and informative. The spec was written from evidence, not theory.

**Stage 4 — Spec**
Write the contract from what you learned. Not before. The design pipeline contract was written after four iterations of the builder, after understanding Ropau's system deeply, after Emil's taste-encoding methodology, after the Google spec format. The contract reflects reality.

### What qualifies a domain

A domain qualifies for the deep process when agent failures in that domain become **specific**, **recurring**, and **consequential**. Generic failures ("the output wasn't great") do not qualify. Specific failures do ("the agent defaults to dark-first backgrounds regardless of the signature", "the agent produces type-unsafe code in this exact pattern when given ambiguous types").

### Current domain status

| Domain | Stage | Contract |
|---|---|---|
| Design | Spec complete | `design-pipeline-contract.md` |
| Frontend (React/TSX) | Lightweight SKILL.md | Pending first real failures |
| TypeScript | Lightweight SKILL.md | Pending first real failures |
| Backend / Express | Lightweight SKILL.md | Pending first real failures |
| Database | Not started | Pending first real failures |
| Testing | Not started | Pending first real failures |
| Wiki operations | Lightweight SKILL.md | Pending first real failures |

---

## 7. Design Pipeline

The design pipeline governs how aesthetic intent travels from a client brief into deployed UI. It is the first fully-specced domain in the system. Full specification: `Bifrost/contracts/design-pipeline-contract.md`.

### Core vocabulary

**Rune** — A named, versioned design identity. Encodes visual DNA, motion system, routing governance, and annotated radar for a specific archetype profile. Lives in `Sanctum/10 Knowledge/Design/runes/`. Reusable across projects sharing the same archetype profile.

**Runestone** — The browsable web interface that renders the Rune library as a live experience. Type specimens, colour swatches, motion previews, radar visualisation, selection guide. Built by the design pipeline itself (dogfood). Phase 1 deliverable.

**Design Contract** — The project-specific, agent-executable file instanced from a Rune for a given project. Lives at project root as `DESIGN_[PROJECT].md`. Complete and self-standing — design/builder requires no other input. Google `design.md` spec compliant in its YAML frontmatter.

**Axes** — Eight visual/perceptual dimensions used to score every Rune. Each axis has a score (1–10) and a one-sentence interpretation specific to that Rune's expression of it.

| Axis | Low | High |
|---|---|---|
| Density | Airy | Dense |
| Temperature | Cold | Warm |
| Luminosity | Dark-first | Light-first |
| Energy | Static | Expressive |
| Geometry | Organic | Geometric |
| Chromatism | Monochrome | Polychrome |
| Depth | Flat | Layered |
| Posture | Utility | Experience |

**Moments** — Named, timed animation events that are part of a Rune's identity. 2–5 per Rune. Each expresses a specific aspect of the archetype through motion. Not decorative — semantic.

**Token format** — OKLCH with two-letter namespace prefixes (`--sb-` Signal Brut, `--nh-` Néon Humaniste, `--ae-` Argent Décisif, `--ee-` Encre Éditoriale). Hex shown as secondary reference only.

### The pipeline stages

| Stage | Skill | Phase | Who | Input → Output |
|---|---|---|---|---|
| 0 — Routing | — | 0 | Human | Brief → Selected Rune |
| 1 — Instancing | design/curator | 0 | Agent | Brief + Rune → Design Contract |
| 2 — Building | design/builder | 0 | Agent | Design Contract → HTML + Run Report |
| 3 — Validation | design/validator | 0 | Agent | HTML + Design Contract → Validation report |
| 4 — Dialogue | design/dialogue | 1 | Agent | Rune + question → Explanation |
| 5 — Harvest | design/harvester | 1 | Agent | Run reports → Harvest report |
| 6 — Distill | design/distill | 2 | Agent | Harvest report → Proposed Rune update |

**Routing is human.** The routing guide in Sanctum has six questions. You answer them. You select the Rune. design/curator instances from your selection — it never selects independently.

### Current Rune library (Phase 0 — to write)

| Rune | Archetypes | Radar | Primary use |
|---|---|---|---|
| Argent Décisif | Ruler + Sage | 9-2-9-1-6-9-8-1 | B2B authority, data tools, professional services |
| Néon Humaniste | Creator + Lover | 4-8-4-7-4-8-6-6 | Creative tools, independent professionals, warm SaaS |
| Signal Brut | Outlaw + Creator | 4-3-7-5-9-6-8-2 | Media monitoring, investigative tools, anti-algo products |
| Encre Éditoriale | Sage + Creator | 6-5-8-2-4-9-9-5 | Publishing, editorial, long-form, institutional |

---

## 8. Agent Taxonomy

Agents are organised into four tiers. Tier determines context width, execution environment, and who may dispatch them.

### Tier 1 — Actions (atomic)

One domain. One verb. ≤10 files. Foreground or background.

**Producers** generate output: code, docs, wiki pages, HTML, run reports.
**Validators** report findings without modifying files.

Rule: validators never fix. They report. A separate producer fixes.

### Tier 2 — Workflows (lifecycle chains)

Multiple actions chained within a domain. Covers a complete lifecycle unit.

**Routine (background):** `react-feature`, `api-endpoint`, `wiki-maintain`, `design-run`
**Complex (foreground):** `ddd-bounded-context`, `bdd-cycle`

### Tier 3 — Projects (cross-domain)

Multiple workflows across domains. Always foreground. Always human-initiated.

`full-stack-feature`, `wiki-build`, `event-sourced-module`, `runestone-build`

### Tier 4 — Strategy

You. Full context. All routing decisions.

---

## 9. Hardware Topology

| Node | Role | Models |
|---|---|---|
| MBP | Foreground: Claude Code, Ghostty/Tmux | Claude Sonnet / Opus |
| Mac mini M1 (16GB) | Background: heavy runs, Ollama, GitHub CLI | GLM-4.7-Flash or Qwen2.5-Coder:7b locally; Kimi K2.6 via API for complex tasks |
| Particle Tachyon (QCM6490, 8GB, Ubuntu 24.04, 5G) | Gateway + lightweight background: webhook listener, routing | Phi-3-mini / Qwen2.5-Coder:1.5b |

### Mac mini role

Runs Claude Code in named tmux sessions via SSH. Controlled externally via SSH + tmux + Tailscale. Does not run OpenClaw — see note below.

### Tachyon role

Dispatch layer. Receives Slack commands. SSH-es into Mac mini to launch tmux sessions. Runs lightweight dispatch routing locally. Posts results back to Slack.

### External control

Primary: SSH + tmux + Tailscale. Secondary: Slack webhook via Tachyon. No third-party agent-as-a-service in the critical path.

**On OpenClaw:** Do not build around it. Anthropic closed the Max license path for external agents. OpenAI acquired it. Costs are uncontrolled at scale. The autonomy model conflicts with the human-orchestrator principle. SSH + tmux + Tailscale gives equivalent capability with full observability and no vendor dependency.

---

## 10. Model Routing

| Task class | Model | Node |
|---|---|---|
| Complex reasoning, architecture, design curation | Claude Opus (latest) | MBP |
| Standard feature work, code generation | Claude Sonnet (latest) | MBP |
| Background heavy: multi-file refactor, wiki ingest | GLM-4.7-Flash (local) or Kimi K2.6 (API) | Mac mini |
| Dispatch routing, lightweight classification | Phi-3-mini | Tachyon |
| Mechanical production: rename, format, stub | Qwen2.5-Coder:1.5b | Tachyon |

### Model selection rules

- Never use a weaker model for creative or architectural decisions. Quality degradation is invisible until it causes a problem.
- GLM-4.7-Flash (9B active, 128K context, strong tool calling) fits Mac mini 16GB with Chrome and other processes running.
- Kimi K2.6 via Moonshot API (`moonshotai/kimi-k2.6`, OpenAI-compatible, $0.60/$2.50 per M tokens) for complex background tasks requiring stronger reasoning.
- Kimi K2.6 cannot run locally — requires ~256GB combined RAM/VRAM.

---

## 11. Interface Layer

### Claude Code (primary)

The main foreground interface. Used for complex tasks: architecture decisions, design curation, code review, anything requiring judgment. Reads `AGENTS.md` and `SKILL.md` on startup.

### Slack (ambient dispatch)

Channel structure follows agent taxonomy. Dispatch commands routed through Tachyon. Results posted back as links (PR, run report, Inbox file).

### Runestone (design)

The Runestone (`localhost:3100` in development, deployed to production in Phase 1) is the interface to the Rune library. Used before every new design project run: navigate to `/guide`, answer six questions, select a Rune, dispatch design/curator.

---

## 12. AGENTS.md Contract

Every repository that agents operate in has an `AGENTS.md` at the root. The file is the operating contract between you and every agent that reads it.

### Required sections

```markdown
# AGENTS.md

## Identity
What this repo is. Who maintains it. What agents are authorised to do.

## Authorised actions
What agents may and may not do. File write permissions. Branch creation rules.

## Quality gate
How to run the quality gate. What exit codes mean.

## Skill loading
Which Bifrost skills are loaded for this repo. Path to skill index.

## Context boot
Files agents must read before beginning work. Order matters.

## Feedback channel
Where agents write feedback and suggestions. Always `00 Inbox/harness-feedback.md`.
```

### The context boot principle

Agents do not arrive with context. `AGENTS.md` defines what to read first, in order. A well-written context boot means the agent begins work already knowing: the project's architecture, the quality standards, the relevant skills, and the current task context. A poorly written context boot means the agent makes assumptions.

---

## 13. SKILL.md Contract

Every skill in Bifrost is a directory containing a single `SKILL.md`. The directory name is the skill identifier. Skills are the executable unit of knowledge in the system.

### Required fields

```yaml
---
name: design/curator
version: 1.0.0
type: producer          # producer | validator | workflow
domain: design
tier: action            # action | workflow | project
triggers:
  - /design-curator
  - "instance a design contract"
reads_from: Sanctum/10 Knowledge/Design/runes/[slug].md
writes_to: DESIGN_[PROJECT].md
model_preference: claude-sonnet
---
```

### Required sections

**When to use** — The conditions that trigger this skill. Specific enough that an agent can determine whether to load it.

**What it does** — Numbered steps. What the skill produces. What side effects it has.

**What it does NOT do** — Explicit non-scope. Prevents agents from expanding scope under ambiguity.

**Inputs** — Required and optional. Types and formats.

**Output** — What the skill writes, to where, in what format.

**Failure modes** — The 2–4 most common failure patterns and how to handle each.

---

## 14. GitHub Skills

GitHub skills operate via the `gh` CLI. They follow the same validator/producer split as other skills.

**Read skills (Phase 1):**
- `github/read-file` — read a specific file at a ref
- `github/read-diff` — read the diff for a PR or commit range
- `github/read-history` — read commit log for a path
- `github/read-pr` — read PR description, comments, and review state

**Write skills (Phase 4):**
- `github/create-branch` — create a branch from a base ref
- `github/commit-file` — write a file to a branch
- `github/open-pr` — open a draft PR with description
- `github/comment-pr` — add a comment to a PR

Write skills require loop-back validation before use. All agent PRs open as draft. Human marks ready.

---

## 15. Quality Gate

The quality gate is a shell script at `Bifrost/scripts/quality-gate.sh`. It has two modes:

**`--llm-only` (fast, <30s):** Runs lint with the LLM lint surface + TypeScript type check. This is the gate agents run in the loop-back. Produces structured JSON output.

**`--full`:** Runs format + lint + typecheck + tests + build + changeset check. Runs on pre-push and in CI.

### LLM lint surface

A dedicated ESLint config (`eslint.llm.config.mjs`) that promotes agent-relevant warnings to errors. Agents run `npm run lint:llm` which uses this config. Human-facing lint uses the standard config.

The LLM lint surface catches: missing return types, any assertions, unused variables, type-unsafe patterns common in agent-generated code.

---

## 16. Loop-back Gate

The loop-back gate wraps any agent invocation and enforces quality before output reaches a human.

```
Agent runs → quality-gate.sh --llm-only
  Exit 0: agent output moves to Inbox for human review
  Exit 1: structured feedback JSON → agent retry (max 3x)
    After 3 failures: escalate to human with failure JSON
```

**Implementation:** `Bifrost/scripts/loop-back.sh` — ~50 lines. Captures exit code and structured JSON from quality gate. Formats feedback object. Re-invokes agent with feedback prepended to prompt. Tracks retry count. Escalates after max retries.

**Built in Phase 2.** Prerequisite: at least one working producer skill to test against.

---

## 17. Promote Gate + Self-Improvement Loop

### Promote gate

Nothing moves from `00 Inbox/` to durable wiki without human review. The gate is:

1. Agent writes output to `00 Inbox/[skill-id]-[date].md`
2. Human opens the file, reads it
3. Human approves: moves file to target location in `10 Knowledge/` or `30 Memory/`
4. Human rejects: discards file, optionally annotates `harness-feedback.md`
5. Human partially approves: edits file inline before promoting

### Self-improvement loop

The system proposes its own improvements and you decide what to commit.

```
Run output → design/harvester (or wiki/lint, etc.)
  → Harvest report in 00 Inbox/
  → Human review
  → Approved findings → design/distill (or wiki/ingest, etc.)
  → Proposed updates in 00 Inbox/
  → Human commits changes to Rune / wiki / SKILL.md
```

The system never commits improvements to itself. Human always commits. This is the same principle as the promote gate applied to self-modification.

---

## 18. The Core Operating Loop

### Wiki operations

`wiki/ingest`: source → summary page + index update + log entry. One source touches 10–15 pages.

`wiki/query`: question → answer + (if durable) file back as wiki page.

`wiki/lint`: periodic health check → contradictions, orphans, stale claims → Inbox report.

### Design operations

`design/curator`: brief + rune selection → Design Contract.

`design/builder`: Design Contract → HTML + Run Report → Inbox.

`design/validator`: HTML + Design Contract → Validation report → Inbox.

`design/harvester`: Run reports → Harvest report → Inbox.

### Heartbeat (Tachyon, Phase 5+)

Cron-driven. Every 6h: `wiki/lint`. Every 24h: `docs/drift` + `index/rebuild`. Weekly: stale-scanner. Produces digest to `00 Inbox/` + Slack notification if actionable.

---

## 19. Phased Build Sequence

### Phase 0 — Foundation (current priority)

**Status:** Ready to build

**System infrastructure:**
- Create `sanctum/` private git repo
- Create `bifrost/` public git repo
- Scaffold both folder structures per spec
- Create `Sanctum/01 Indexes/log.md` (empty, present from day one)
- Create all templates in `Sanctum/99 Templates/`
- Symlink `bifrost/skills/` → `~/.agents/skills/bifrost/`
- Install Ollama on Mac mini + first model running
- Pi on Mac mini — manual dispatch proven once

**Design domain:**
- Write four core Rune files in Sanctum (Argent Décisif, Néon Humaniste, Signal Brut, Encre Éditoriale)
- Write `routing-guide.md` in Sanctum
- Write `design-principles.md` in Sanctum (Emil's motion grammar, axis definitions)
- Write `design/curator` SKILL.md in Bifrost
- Write `design/builder` SKILL.md in Bifrost
- Write `design/validator` SKILL.md in Bifrost
- Ingest this conversation's session summary into Sanctum `40 Sources/`

**Dogfood:** design pipeline runs on the Runestone brief to produce its Design Contract.

---

### Phase 1 — Runestone + GitHub Read

**Status:** Not yet started

**Design:**
- Build Runestone v1 — first real design pipeline project run
- Runestone deployed to production
- `design/dialogue` SKILL.md written
- `design/harvester` SKILL.md written
- First harvest report from Runestone build run

**GitHub:**
- `github/read-file` SKILL.md
- `github/read-diff` SKILL.md
- `github/read-history` SKILL.md
- `github/read-pr` SKILL.md

**Dogfood:** GitHub read skills are committed to Bifrost. Agents can read the commit that added them.

---

### Phase 2 — First Skill + Loop-back

**Status:** Not yet started

Prove the quality loop closes. Most important correctness milestone.

- `docs/changelog` SKILL.md — reads diff, writes changelog entry
- `docs/drift` SKILL.md — checks code/docs alignment
- Loop-back wrapper (`Bifrost/scripts/loop-back.sh` — ~50 lines)
- `design/distill` SKILL.md written

**Dogfood:** `docs/changelog` writes its own changelog entry after being built. Loop-back runs on both changelog and drift. Distill runs on first harvest report from Runestone.

---

### Phase 3 — Wiki Core Loop

**Status:** Not yet started

Sanctum becomes a real compiling wiki.

- `wiki/ingest` SKILL.md
- `wiki/query` SKILL.md
- `wiki/lint` SKILL.md
- Ingest this system's own design documents as first real sources
- Run first lint pass

**Dogfood:** `wiki/ingest` files the notes from its own creation session. The system begins compounding.

---

### Phase 4 — GitHub Write Skills

**Status:** Not yet started

Agents materialise output as PRs.

- `github/create-branch` SKILL.md
- `github/commit-file` SKILL.md
- `github/open-pr` SKILL.md (always draft; loop-back must pass first)

**Dogfood:** `docs/changelog` opens its first real PR on the changelog for adding the write skills.

---

### Phase 5 — Slack + Tachyon

**Status:** Not yet started

The system becomes ambient.

- Tachyon: Ollama running (1–3B ARM models)
- Tachyon: Pi installed and tested
- Slack workspace — channel structure per taxonomy
- Tachyon: webhook listener wired to Slack
- Tachyon → Mac mini routing for heavy tasks

**Dogfood:** Dispatch `docs/changelog` from phone via Slack. Opens PR. Posts link back to Slack. First fully ambient loop.

---

### Phase 6 — Self-Improvement Loop

**Status:** Not yet started

The promote gate gets tooling. The system stores memory of building itself.

- `memory/write` skill (session-promoter)
- `github/comment-pr`, `github/request-review`, `github/mark-ready` SKILL.md
- `index/rebuild` skill
- Backfill: all session summaries from Phases 0–5 ingested into Sanctum
- Harness-feedback review workflow established

**Dogfood:** `memory/write` promotes its own build session into a memory note.

---

### Phase 7 — Full Workflow Chains

**Status:** Not yet started

Full workflow chains. Real product work begins.

- `react-feature` workflow
- `api-endpoint` workflow
- `type-safety` workflow
- Model routing rules active — Mac mini vs Tachyon by task class

**Dogfood:** `react-feature` workflow builds the Runestone's interactive motion preview components.

---

### Phase 8 — Webhook Triggers

**Status:** Not yet started

Pull becomes push for proven roles (≥2 weeks predictable pull-mode).

- `github/on-ci-failure` — dispatches bug-find on CI fail
- `github/on-pr-open` — dispatches review workflow on PR open
- `github/on-push-main` — dispatches changelog and drift on push to main

**Dogfood:** CI failure on the system's own repo triggers a bug-find action.

---

## 20. Conventions

| Convention | Format | Examples |
|---|---|---|
| Skill IDs | `domain/action` kebab-case | `design/curator` · `wiki/ingest` · `docs/changelog` |
| Workflow IDs | `domain-lifecycle` | `react-feature` · `design-run` · `wiki-maintain` |
| Rune slugs | kebab-case | `signal-brut` · `argent-decisif` · `neon-humaniste` |
| Rune namespaces | two-letter prefix | `--sb-` · `--ae-` · `--nh-` · `--ee-` |
| Token format | OKLCH + namespace | `--sb-accent-yellow: oklch(95% 0.25 110)` |
| Design Contract names | `DESIGN_[PROJECT].md` | `DESIGN_SIGNAL.md` · `DESIGN_RUNESTONE.md` |
| Branch — human | `feat/*` · `fix/*` · `chore/*` · `docs/*` | `feat/runestone-hero` |
| Branch — agent | `bg/*` · `skill/*` | `bg/runestone-sections` · `skill/design-curator` |
| Tmux session names | `[role]:[task-slug]` | `design-worker:runestone-build` |
| Session summaries | `[project]-[YYYY-MM-DD].md` | `design-pipeline-session-2026-04-21.md` |
| Inbox files | `[skill-id]-[date].md` | `design-builder-2026-04-22.md` |
| Run reports | `run-report-[date].md` | `run-report-2026-04-22.md` |
| Gate failure files | `gate-failure-[branch].md` | `gate-failure-bg-runestone-build.md` |
| Log entries | `## [YYYY-MM-DD] op \| detail` | `## [2026-04-22] design/curator \| signal-brut → DESIGN_SIGNAL.md` |
| Commit format | Conventional Commits | `feat(design): add signal-brut rune file` |
| Rune version | semver | `1.0.0` · `1.1.0` · `2.0.0` |
| Memory confidence | `verified` · `inferred` | `confidence: inferred` |
| Note status | `evergreen` · `working` · `archived` | `status: working` |

---

*End of system-spec v0.3*
