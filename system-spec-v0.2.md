# Sanctum + Bifrost — System Specification

**Version:** 0.2  
**Status:** Design complete. Ready to build.  
**Sanctum repo:** private  
**Bifrost repo:** public  
**Last updated:** 2026-04-20

---

## Table of Contents

1. [Overview](#1-overview)
2. [Core Principles](#2-core-principles)
3. [Build Status](#3-build-status)
4. [Sanctum — The Compiled Wiki](#4-sanctum--the-compiled-wiki)
5. [Bifrost — The Schema Layer](#5-bifrost--the-schema-layer)
6. [Agent Taxonomy](#6-agent-taxonomy)
7. [Hardware Topology](#7-hardware-topology)
8. [Model Routing](#8-model-routing)
9. [Interface Layer](#9-interface-layer)
10. [AGENTS.md Contract](#10-agentsmd-contract)
11. [SKILL.md Contract](#11-skillmd-contract)
12. [GitHub Skills](#12-github-skills)
13. [Quality Gate](#13-quality-gate)
14. [Loop-back Gate](#14-loop-back-gate)
15. [Promote Gate + Self-Improvement Loop](#15-promote-gate--self-improvement-loop)
16. [The Core Operating Loop](#16-the-core-operating-loop)
17. [Phased Build Sequence](#17-phased-build-sequence)
18. [Conventions](#18-conventions)
19. [Existing Infrastructure](#19-existing-infrastructure)

---

## 1. Overview

This document specifies a hybrid human-orchestrated agent system. It has two shared layers and any number of project consumers.

**Sanctum** is the compiled knowledge wiki. It owns everything the system knows — sources, synthesis, memory, indexes, project context. It is maintained by agents but read by both humans and agents. Knowledge compounds here over time.

**Bifrost** is the schema and operating system that maintains the wiki. It owns skills, contracts, routing rules, verification logic, and prompt assembly rules. It tells agents how to operate. Projects consume both.

The orchestrator is a human. Automation is earned, not assumed.

The primary product being built with and on this system is **FODMAPP**. The system dogfoods itself — agents maintain the wiki that guides the agents that build the product.

### What this system is not

Not an automated orchestration framework. No AI orchestrator. No agents spawning agents without human dispatch. The system is a set of well-defined scoped agents that a human routes between. Intelligence lives at the top, not in a routing layer.

### The Karpathy model

This system is a direct implementation of Karpathy's LLM Wiki pattern, generalised into a broader agent OS:

- **Raw sources** (Sanctum `40 Sources/`) — immutable inputs the LLM reads but never modifies
- **Compiled wiki** (Sanctum `10 Knowledge/`, `01 Indexes/`, `30 Memory/`) — LLM-maintained synthesis that compounds over time
- **Schema** (Bifrost) — the operating instructions and skills that tell the LLM how to maintain the wiki

The key property: the wiki is a **persistent compounding artifact**. Cross-references are already there. Contradictions have been flagged. Synthesis reflects everything ingested. The wiki gets richer with every source added and every question asked. Good answers get filed back as wiki pages. Nothing useful disappears into chat history.

---

## 2. Core Principles

### Human orchestrator
The human makes all routing decisions, task decompositions, and promote gate judgments. No agent decides which agent runs next. Automation is for execution, not orchestration.

### Sources are immutable
Raw source material in `40 Sources/` is never modified by agents. The LLM reads from it but writes only to the compiled wiki zone. Provenance must be traceable.

### The wiki compounds
Every useful synthesis — an answer, an analysis, a connection discovered — gets filed back into the wiki. Nothing valuable disappears into chat history. The system gets smarter with every interaction.

### Bifrost is the schema, not the knowledge
Bifrost defines how the wiki is maintained. Sanctum contains what the wiki knows. These are different concerns and must never blur.

### Harness agnosticism
`AGENTS.md` and `SKILL.md` are the harness contract. Pi, Claude Code, and Codex all read them. The system does not depend on any single harness.

### Pull before push
Every automated trigger starts as manual dispatch. A role earns automated triggering only after ≥2 weeks of predictable manual use. Automation is a reward for proven reliability.

### Dogfood the build
Each phase uses what was built in the previous phase. The system builds itself. The changelog skill writes its own changelog. The wiki/ingest skill files the notes from its own creation.

### Full observability
Every gate check is a command you can run yourself. Every agent output is a file you can read. Every failure is structured output you can inspect.

### Promote gate
Nothing moves from agent output to durable wiki without human review. Background agents write to `00 Inbox/` only. You review and approve before anything reaches `10 Knowledge/` or `30 Memory/`.

### Narrow context
Action agents operate on ≤10 files. Workflow agents load one domain map. Project agents load cross-domain maps. Strategy agents (you) hold full context. More context than needed causes scope creep and costs more in both tokens and correctness.

---

## 3. Build Status

| Capability | Status | Location |
|---|---|---|
| Root AGENTS.md (FODMAPP) | ✅ built | `fodmapp/AGENTS.md` |
| LLM lint surface | ✅ built | `fodmapp/eslint.llm.config.mjs` |
| CLI quality gate | ✅ built | `fodmapp/.github/scripts/quality-gate.sh` |
| Pre-push enforcement | ✅ built | `fodmapp/.githooks/pre-push` |
| Agent branch identity (`codex/*`) | ✅ built | `fodmapp/CONTRIBUTING.md` |
| Worktree model | ✅ built | `fodmapp/docs/ops/worktree-status.md` |
| FODMAPP skills (Turborepo) | ✅ built | `fodmapp/skills/` — 3 skills, prior art for format |
| Sanctum repo | ❌ not yet | Phase 0 |
| Bifrost repo | ❌ not yet | Phase 0 |
| Sanctum folder structure | ❌ not yet | Phase 0 |
| Bifrost folder structure | ❌ not yet | Phase 0 |
| `log.md` in Sanctum | ❌ not yet | Phase 0 |
| Loop-back wrapper | ❌ not yet | Phase 2 — ~50 lines |
| `docs/changelog` skill | ❌ not yet | Phase 2 — first skill |
| `wiki/ingest` skill | ❌ not yet | Phase 3 — proves compounding loop |
| `wiki/query` skill | ❌ not yet | Phase 3 |
| `wiki/lint` skill | ❌ not yet | Phase 3 |
| GitHub read skills | 🟡 partial | gh CLI present, not yet SKILL.md |
| GitHub write skills | ❌ not yet | Phase 4 |
| Slack gateway | ❌ not yet | Phase 5 |
| Ollama on Mac mini | ❌ not yet | Phase 0 prerequisite |
| Tachyon configured | ❌ not yet | Phase 5 |
| Session-promoter agent | ❌ not yet | Phase 6 |

---

## 4. Sanctum — The Compiled Wiki

Sanctum is a git-backed private repository. It is the system's long-term memory, research archive, and knowledge base. It has two zones with different mutability rules.

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
  system-spec.md (symlink to bifrost/system-spec.md)

  00 Inbox/
    harness-feedback.md          ← agents append AGENTS.md improvement suggestions

  01 Indexes/
    index.md                     ← master navigation entry point
    log.md                       ← append-only chronological record of ingests/queries/lint
    knowledge-map.md
    memory-map.md
    projects-map.md
    sources-map.md

  10 Knowledge/
    Articles/
    Concepts/
    Lessons/
    Exercises/

  20 Projects/
    _project-template.md
    project-index.md

  30 Memory/
    Environment/
    Preferences/
    Project Memory/

  40 Sources/                    ← IMMUTABLE — LLM reads, never writes
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
```

Note: `50 Outputs/` is intentionally absent. Transient outputs land in `00 Inbox/` and are either promoted to `10 Knowledge/` or discarded. There is no permanent intermediate layer — per Karpathy: good answers become wiki pages.

### `log.md` — first-class maintenance file

An append-only chronological record of system operations. Every ingest, query, and lint pass gets an entry. Parseable with simple Unix tools.

Format:
```
## [2026-04-20] ingest | Karpathy - LLM Wiki
## [2026-04-20] query | How does prompt caching work in Claude Code?
## [2026-04-21] lint | Found 3 orphan pages, 1 stale claim
```

### Access contract

| Path | Read | Write | Rule |
|---|---|---|---|
| `00 Inbox/` | you | bg agents | All bg output lands here. Nothing leaves without your review. |
| `01 Indexes/` | all agents | wiki agents | Navigation layer. `log.md` is append-only by agents. |
| `10 Knowledge/` | all agents | wiki agents (via gate) | Primary compiled content. Updated via promote gate only. |
| `20 Projects/` | all agents | foreground agents | Project context. Foreground reads/writes freely. |
| `30 Memory/` | all agents | you only | Gate-protected. Never written by background agents directly. |
| `40 Sources/` | all agents | you only | Immutable. Agents read, never write. |
| `60 Assets/` | all agents | you only | Supporting files. |
| `90 Archive/` | all agents | you | Deprecated material. |
| `99 Templates/` | all agents | you | Schemas. Not modified by agents. |

### Note templates

| Template | Frontmatter | Sections |
|---|---|---|
| `concept-note.md` | type, status, updated, tags | Summary, Why it matters, Related |
| `memory-note.md` | type, scope, confidence, updated | Verified, Inferred |
| `project-note.md` | type, project, status, updated | Context, Decisions, Constraints, Related |
| `session-summary.md` | type, agent, project, date, promote_to_memory | Objective, What happened, Decisions, Follow-up |
| `source-note.md` | type, source_kind, author, captured | Summary, Relevance, Extracted ideas |
| `research-note.md` | type, topic, confidence, updated | Question, Findings, Sources, Open questions |

---

## 5. Bifrost — The Schema Layer

Bifrost is a git-backed public repository. It is the operating system that tells agents how to maintain the wiki, how to execute tasks, and how to verify their work. It owns skills, contracts, routing, and scripts.

**Bifrost is not the knowledge base. It is the operating system for the knowledge base.**

### Folder structure

```
bifrost/
  README.md
  AGENTS.md
  system-spec.md

  skills/
    _index.md                    ← human-readable skill inventory
    _template/
      SKILL.md                   ← canonical skill template
    wiki/
      wiki-ingest/
        SKILL.md
      wiki-query/
        SKILL.md
      wiki-lint/
        SKILL.md
    docs/
      docs-changelog/
        SKILL.md
      docs-write/
        SKILL.md
      docs-drift/
        SKILL.md
      docs-adr/
        SKILL.md
    react/
      react-implement/
        SKILL.md
      react-test/
        SKILL.md
      react-review/
        SKILL.md
      react-bug-find/
        SKILL.md
      react-perf-audit/
        SKILL.md
    typescript/
      ts-type/
        SKILL.md
      ts-audit/
        SKILL.md
    express/
      express-route/
        SKILL.md
      express-schema/
        SKILL.md
    github/
      gh-read-file/
        SKILL.md
      gh-read-diff/
        SKILL.md
      gh-create-branch/
        SKILL.md
      gh-open-pr/
        SKILL.md
      gh-comment-pr/
        SKILL.md
    memory/
      memory-write/
        SKILL.md
      memory-summarize/
        SKILL.md
    verification/
      verify-output/
        SKILL.md
    architecture/
      arch-solid/
        SKILL.md
      arch-ddd-review/
        SKILL.md
    security/
      sec-audit/
        SKILL.md
      sec-auth-review/
        SKILL.md

  contracts/
    skill-contract.md
    memory-contract.md
    prompt-contract.md
    verification-contract.md
    project-boundaries.md

  scripts/
    loop-back.sh                 ← feeds gate failure JSON back to agent
    quality-gate.sh              ← thin wrapper calling project gates
    skill-lint.sh                ← validates SKILL.md files against contract

  examples/
    skill-example.md
    agent-flow-example.md
```

### Skill discovery

Skills are discovered via the standard `~/.agents/skills/` global path that all major harnesses (Pi, Claude Code, Codex, OpenCode) scan automatically. Bifrost's skills directory is symlinked there. No custom registry file needed — native harness discovery handles it.

```bash
# On setup
ln -s ~/Documents/bifrost/skills ~/.agents/skills/bifrost
```

Each harness loads skill metadata (name + description) at startup. Full SKILL.md body loads only when the skill is activated. This is progressive disclosure — context-efficient by design.

---

## 6. Agent Taxonomy

Four tiers. The distinguishing property is scope of judgment and context budget. Domain verticals cut through all tiers.

---

### Tier 1 — Actions

> One domain × one verb · ≤10 files · deterministic output · background tier

**Producers** generate artifacts. **Validators** inspect artifacts. Strict separation — producers never validate, validators never fix.

#### Producer actions

| ID | Input | Output |
|---|---|---|
| `react/implement` | component spec | component file(s) |
| `react/test` | component file | test suite |
| `react/story` | component file | Storybook stories |
| `ts/type` | shape / runtime value | type definitions |
| `ts/narrow` | unsafe branches | narrowed branches |
| `express/route` | route spec | handler file |
| `express/schema` | payload shape | Zod/Joi schema |
| `docs/write` | code or spec | documentation file |
| `docs/changelog` | git diff (via `github/read-diff`) | changelog entry → PR |
| `docs/adr` | decision brief | Architecture Decision Record |
| `rag/chunk` | raw documents | chunked documents |
| `rag/embed` | chunked documents | embeddings stored |
| `design/token` | design brief | token files (DTCG) |
| `memory/write` | session summary + target note | draft memory update → Inbox PR |
| `memory/summarize` | session or context | compact summary |
| `wiki/ingest` | source file | updated wiki pages + index + log entry |
| `wiki/query` | question | synthesised answer + filed wiki page |
| `wiki/lint` | wiki state | health report → Inbox |
| `index/rebuild` | vault folder scan | updated map files + dead-link report |

#### Validator actions

| ID | Input | Output |
|---|---|---|
| `react/review` | diff or PR | review report → Inbox |
| `react/bug-find` | failing file(s) | bug report → Inbox |
| `react/perf-audit` | component + render profile | perf report → Inbox |
| `ts/audit` | module files | type safety report → Inbox |
| `arch/solid` | module files | SOLID audit → Inbox |
| `arch/ddd-review` | domain model files | DDD audit → Inbox |
| `arch/event-review` | event model files | event sourcing audit → Inbox |
| `sec/audit` | code surface | vulnerability report → Inbox |
| `sec/auth-review` | auth/middleware files | auth model audit → Inbox |
| `rag/quality` | retrieval results + ground truth | quality report → Inbox |
| `design/token-validate` | token files | parity report → Inbox |
| `design/a11y-audit` | component markup | a11y report → Inbox |
| `docs/drift` | code files + docs | drift report → Inbox |

---

### Tier 2 — Workflows

> Single domain · full lifecycle · chains actions · background (routine) or foreground (complex)

Validators in chains are marked with `⟨⟩`.

| ID | Domain | Chain | Tier |
|---|---|---|---|
| `react-feature` | React | implement → test → story → ⟨review⟩ → ⟨perf-audit⟩ → docs/write | bg routine |
| `api-endpoint` | Express | route → schema → ⟨auth-review⟩ → ⟨sec/injection⟩ → docs/write | bg routine |
| `type-safety` | TypeScript | ts/type → ts/narrow → ⟨ts/audit⟩ | bg routine |
| `rag-pipeline` | RAG | chunk → embed → retrieve → ⟨rag/quality⟩ | bg routine |
| `design-system` | Design | token → spec → ⟨token-validate⟩ → ⟨contrast⟩ → ⟨a11y-audit⟩ | bg routine |
| `wiki-maintain` | Wiki | ingest → ⟨lint⟩ → index/rebuild | bg routine |
| `ddd-bounded-context` | Architecture/DDD | docs/adr → schema → ⟨arch/ddd-review⟩ → ⟨arch/solid⟩ | fg complex |
| `bdd-cycle` | BDD | docs/write(spec) → react/test(stubs) → react/implement → ⟨react/review⟩ | fg complex |

---

### Tier 3 — Projects

> Multiple domains · single goal · coordinates workflows · foreground tier

| ID | Workflows | Notes |
|---|---|---|
| `full-stack-feature` | react-feature + api-endpoint + type-safety | + sec/audit + docs/changelog on completion |
| `event-sourced-module` | ddd-bounded-context + api-endpoint | arch/event-review injected at project level |
| `search-product` | rag-pipeline + react-feature | + docs coverage on all public surfaces |
| `wiki-build` | wiki-maintain × N sources | systematic wiki construction from a source corpus |

---

### Tier 4 — Strategy

> Product / roadmap scope · full context · you are the primary agent

| ID | Description |
|---|---|
| `architecture-decision` | Evaluates options across full system. Invokes arch validators. Produces ADR. |
| `product-ideation` | Broad exploration. No fixed agent chain. You orchestrate. Produces decision memo. |
| `codebase-evolution` | Multi-sprint refactor plan. Decomposes into sequenced Projects. |
| `system-audit` | Full validator sweep. Produces prioritised remediation backlog. |

---

## 7. Hardware Topology

### MacBook Pro
**Role:** Foreground tier

| Property | Value |
|---|---|
| Tools | Claude Code, Codex App, Ghostty, Tmux |
| Models | Claude Sonnet / Opus, Codex (OpenAI) |
| Agent tiers | Strategy, Projects, Complex Workflows |
| Sanctum | Primary read/write, vault-sync → GitHub |

### Mac mini M1
**Role:** Background tier — heavy workers

| Property | Value |
|---|---|
| Tools | Pi workers, Tmux named sessions, Ollama, GitHub CLI |
| Models | Qwen2.5-Coder-32B, Mistral-Small, Claude API (fallback) |
| Agent tiers | Routine Workflows, Heavy Validators, Mechanical Actions |
| Connectivity | Local network only — SSH accessible |

### Particle Tachyon
**Role:** Background tier — gateway + lightweight workers

| Property | Value |
|---|---|
| Hardware | Qualcomm QCM6490, 8GB RAM, 128GB, Ubuntu 24.04, 12 TOPS NPU |
| Tools | Pi workers, Slack webhook listener, Ollama (ARM), GitHub CLI |
| Models | Phi-3-mini, Qwen2.5-Coder-1.5B |
| Agent tiers | Dispatch router, Lightweight Actions, Watcher agents, Webhook triggers |
| Connectivity | 5G EtherSIM (independent), WiFi 6E |

### Key property: The Tachyon is the gateway

The Mac mini is only reachable on the local network. The Tachyon has an independent 5G connection — reachable from anywhere. It receives all Slack dispatch and GitHub webhooks, routes heavy tasks to the Mac mini, handles lightweight actions locally. It is the always-on ambient interface layer.

---

## 8. Model Routing

> Start with everything on Claude. Migrate as each node comes online with evidence from real runs.

| Task class | Model | Node |
|---|---|---|
| Strategy · Planning | Claude Opus / Sonnet | MBP |
| Complex Workflows | Claude Sonnet | MBP |
| Routine Workflows | Ollama 7B+ (Qwen2.5-Coder-32B) | Mac mini |
| High-stakes Validators | Claude Sonnet | MBP |
| Routine Validators | Ollama 7B+ | Mac mini |
| Mechanical Producers | Ollama 1–3B (Phi-3-mini) | Tachyon |
| Dispatch + routing | Node.js — no model | Tachyon |

**Migration sequence:** P0–P3: all Claude → P4: mini handles routine → P5+: Tachyon handles mechanical → P7: full matrix active.

**Escalation rule:** If an OSS model's output consistently fails your review or the loop-back gate, escalate that class to Claude. Write the exception into AGENTS.md routing section.

---

## 9. Interface Layer

### Foreground — Claude Code and Codex Apps (MBP)
Active pairing. High-attention, back-and-forth judgment. Sessions always end with a session summary in `Sanctum/00 Inbox/` with a `promote_to_memory` gate.

### Background — Ghostty + Tmux (Mac mini + Tachyon)
Named sessions per role. Dispatch and check back. Session names: `[role]:[task-slug]` e.g. `react-worker:ingredient-search`.

### Ambient — Slack (Tachyon gateway)

```
#dispatch          ← task dispatch from any device
#inbox-review      ← all agent output for your approval
#wiki-worker       ← ingest, query, lint operations
#react-worker      ← react-feature workflow
#api-worker        ← api-endpoint workflow
#docs-worker       ← changelog, drift, docs/write
#maintenance       ← stale scanner, index-rebuilder
#harness-feedback  ← agents post AGENTS.md improvement suggestions
```

### Trigger model

**Pull now.** Every role starts in pull mode — you dispatch manually.

**Push later (Phase 7).** Roles dispatched predictably for ≥2 weeks earn automated triggers. Qualification: your judgment adds nothing to the trigger decision.

---

## 10. AGENTS.md Contract

Each layer has its own AGENTS.md. Root AGENTS.md in each repo covers the layer's conventions. Projects have their own thin AGENTS.md pointing at shared skills.

**Size constraint:** Root AGENTS.md: under 200 lines. Scoped (per-role) AGENTS.md: under 80 lines.

### Sanctum AGENTS.md template

```markdown
# AGENTS.md — Sanctum

## Role
Sanctum is the compiled knowledge wiki. Read it like a codebase.
The LLM maintains the compiled zone. Humans maintain the source zone.

## Zone Rules
- 40 Sources/ is IMMUTABLE. Read only. Never write.
- Everything else is the compiled zone. LLM writes here via the promote gate.

## Context Boot Sequence
1. Read 01 Indexes/index.md first.
2. Load the relevant domain map or project note.
3. Load memory notes only if the task requires them.
4. Do not load lessons or full concept files unless depth is needed.

## Writing Rules
- Write to the canonical location, not a duplicate.
- Good answers become wiki pages in 10 Knowledge/.
- Use the appropriate template from 99 Templates/.
- Log every ingest/query/lint operation to 01 Indexes/log.md.
- Mark uncertain content explicitly.

## Inbox Rule
All background output lands in 00 Inbox/ first.
Nothing reaches 10 Knowledge/ or 30 Memory/ without human review.

## Session Summary
At the end of every session, write a summary to 00 Inbox/ using
the session-summary template. Default: promote_to_memory: no.
Set to yes only if the session revealed a persistent decision or gap.
```

### Bifrost AGENTS.md template

```markdown
# AGENTS.md — Bifrost

## Role
Bifrost is the schema and operating system for Sanctum.
It defines how the wiki is maintained, not what it contains.

## Context Boot Sequence
1. Read the relevant contract from contracts/ for the task type.
2. Load only the skills needed for the current task.
3. Do not load Sanctum content unless the task requires it.

## Skill Rules
- Skills are in skills/ organised by domain/action.
- Discovered via ~/.agents/skills/bifrost/.
- Project-agnostic by default. Project-specific skills are rare.
- AGENTS.md refers to skills by canonical name only.

## Quality Gate
Run .github/scripts/quality-gate.sh before any PR.
Loop-back runs fast gate (--llm-only) before gh:open-pr is invoked.

## Collaboration Contract
- Short-lived branches from main.
- Conventional Commit format.
- Semantic PR titles.
- Agent branches: bg/* or codex/*.

## Safety Rules
- Never write to Sanctum 30 Memory/ directly.
- Never commit secrets or credentials.
- Never rewrite history on shared branches.

## Feedback Rule
If this AGENTS.md has a gap or error, append a one-line note
to Sanctum/00 Inbox/harness-feedback.md.
```

### Project AGENTS.md template

```markdown
# AGENTS.md — [Project Name]

## Global system
Skills: ~/.agents/skills/bifrost/
Sanctum: ~/Documents/sanctum/
Bifrost contracts: ~/Documents/bifrost/contracts/

## Project-specific context
[Only what differs from the global contract]
[Domain constraints, scoring rules, API contracts, etc.]

## Skill routing
[Any project-specific skill triggers or overrides]
```

---

## 11. SKILL.md Contract

Skills live in Bifrost, organised by domain. One skill per directory. Skills are project-agnostic by default.

```markdown
---
name: docs/changelog
version: 1.0.0
type: producer
domain: docs
tier: action
triggers:
  - /docs-changelog
  - "write the changelog"
  - "document this diff"
gates:
  - quality-gate.sh --llm-only
max_iterations: 3
escalate_on_failure: foreground
model_preference: ollama-7b
node_preference: mini
---

# docs/changelog

## When to use this skill
Use when a git diff exists and a changelog entry needs to be written.
Triggers on: PR completion, explicit request, post-merge hook.

## What it does
Reads the git diff via github/read-diff. Writes a structured
changelog entry in Conventional Commit format.

## What it does not do
Does not decide what to release. Does not bump version numbers.
Does not modify code. Does not open PRs (the harness does that).

## Inputs
- Git diff (provided by github/read-diff)
- Optional: PR title and description

## Steps
1. Read the diff
2. Identify change categories (feat, fix, chore, docs, etc.)
3. Write changelog entry following Keep a Changelog format
4. Output to stdout for harness to capture

## Output
Structured changelog entry in markdown. The harness writes it
to the branch and opens a draft PR.

## Gates
- quality-gate.sh --llm-only must pass before PR opens
- If gate fails: structured feedback returned, retry (max 3x)
- If still failing: escalate to foreground, write to Inbox

## Failure modes
- Diff is empty: report and exit cleanly
- Diff is too large: summarise by package/module
- Ambiguous change type: default to chore, note the ambiguity

## Examples
Input: `feat(ui): add SearchBar component`
Output: `### Added\n- SearchBar component for ingredient search (#42)`
```

**Key property — injectable validators:** Validator skills have no fixed position in a workflow. They slot in after any producer. They never fix — they report. Workflow AGENTS.md specifies where validators are injected.

**Skill naming:** `domain/action` using kebab-case. `docs/changelog`, `react/implement`, `wiki/ingest`. Never project-specific names in shared skills.

---

## 12. GitHub Skills

Four capability tiers. Read before write. Write before PR workflow. Webhooks only after ≥2 weeks pull-mode.

| Tier | Phase | Gate | Skills |
|---|---|---|---|
| **Read** | Phase 1 | None — read-only | `github/read-file` · `github/read-diff` · `github/read-history` · `github/read-pr` |
| **Write** | Phase 4 | You review PR. Always branch + draft. Never main directly. | `github/create-branch` · `github/commit-file` · `github/open-pr` |
| **PR Workflow** | Phase 6 | You approve merge. | `github/comment-pr` · `github/request-review` · `github/mark-ready` |
| **Webhooks** | Phase 7 | ≥2 weeks pull-mode proven. | `github/on-ci-failure` · `github/on-pr-open` · `github/on-push-main` |

**Write skill invariant:** `github/open-pr` is never the first thing an agent calls. Loop-back gate runs first. If gate fails after max iterations, agent escalates to foreground rather than opening a PR.

---

## 13. Quality Gate

`quality-gate.sh` exists in FODMAPP. Same gate will be generalised into Bifrost as a thin wrapper that calls project-specific gates.

### Fast mode (loop-back iterations)
Command: `quality-gate.sh --llm-only`  
Runs: `pnpm lint:llm` + `tsc --noEmit`  
Target: under 30 seconds. Structured JSON output. Designed for agent retry loops.

### Full mode (pre-push + CI)
Command: `quality-gate.sh --full`  
Runs: format + lint (base + LLM) + typecheck + tests + build + changeset validation.  
Already enforced via `.githooks/pre-push` and GitHub Actions CI in FODMAPP.

### LLM lint surface
`eslint.llm.config.mjs` applies stricter rules to agent-produced code — all warnings become errors. `lint:llm:ci` is a hard failure on all `codex/*` and `bg/*` branches.

**Key property:** Every gate check is a command you can run yourself. Reproduce what the agent saw: `pnpm lint:llm 2>&1 | jq`

---

## 14. Loop-back Gate

The missing piece. Everything needed to build it already exists in FODMAPP. The wrapper is ~50 lines.

### Flow

```
1. AGENT       Produces files. Commits to bg/* or codex/* branch.
               Does not push or open a PR yet.

2. GATE        Loop-back runs quality-gate.sh --llm-only.
               Captures exit code and ESLint JSON.
               → Exit 0: proceed to PR.
               → Non-zero: build feedback object, re-invoke agent.

3. AGENT       Receives structured feedback. Fixes exactly the
               reported violations. Re-runs. Bounded at max_iterations.
               → On exhaustion: writes failure log to
                 Sanctum/00 Inbox/gate-failure-[branch].md
                 Sends Slack notification. Does not open a PR.

4. GATE        Exit 0 confirmed. Loop-back invokes github/open-pr
               as a draft. PR description includes gate iteration count.
               Branch is pushed at this point.

5. CI          quality-gate.sh --full runs on PR open. Source of truth.
               lint:llm:ci as hard failure.

6. YOU         Judgment gate. Deterministic checks already passed.
               Review for correctness, architectural fit, intent match.
```

### Feedback object

```json
{
  "gate": "lint:llm",
  "exit": 1,
  "violations": [
    {
      "file": "apps/app/src/SearchBar.tsx",
      "line": 14,
      "rule": "no-explicit-any",
      "message": "Unexpected any. Specify a different type."
    }
  ],
  "iteration": 1,
  "max_iterations": 3
}
```

---

## 15. Promote Gate + Self-Improvement Loop

The mechanism by which agent outputs become durable wiki knowledge.

### Flow

```
1. FOREGROUND  Session ends. Agent writes session-summary to
               Sanctum/00 Inbox/ using session-summary template.
               Default: promote_to_memory: no.

2. YOU         Review session summary.
               → No: session archived.
               → Yes: dispatch memory/write via Slack.

3. BG AGENT    memory/write reads the flagged summary.
               Identifies target note. Drafts update.
               Opens draft PR against Sanctum repo.
               Posts PR link to Slack #inbox-review.

4. YOU         Review PR. Merge if accurate.
               Background agents never commit directly to 30 Memory/.
               All updates arrive as PRs you merge.
```

### Harness feedback loop

Agents append gaps/errors to `Sanctum/00 Inbox/harness-feedback.md`. Review weekly. The `docs/drift` skill drafts AGENTS.md patch PRs from high-confidence items. You merge. The system's instructions improve.

---

## 16. The Core Operating Loop

Based on Karpathy's ingest/query/lint model. These three operations are the heartbeat of Sanctum.

### `wiki/ingest`

Triggered when a new source is added to `40 Sources/`.

```
1. Read the source document
2. Discuss key takeaways (or run silently in background mode)
3. Write a summary page to 10 Knowledge/
4. Update 01 Indexes/index.md with the new entry
5. Update all relevant concept and entity pages
6. Append entry to 01 Indexes/log.md
7. Write gate-passing PR for your review
```

A single source may touch 10–15 wiki pages. This is the compounding operation.

### `wiki/query`

Triggered when you ask a question against the wiki.

```
1. Read 01 Indexes/index.md to find relevant pages
2. Load the most relevant pages
3. Synthesise an answer with citations
4. If the answer is valuable: file it back as a new wiki page
5. Append entry to 01 Indexes/log.md
```

Good answers become wiki pages. The wiki gets richer with every question.

### `wiki/lint`

Triggered periodically (weekly or on demand).

```
1. Scan for contradictions between pages
2. Find stale claims newer sources have superseded
3. Identify orphan pages with no inbound links
4. Flag concepts mentioned but lacking their own page
5. Suggest missing cross-references
6. Report data gaps that could be filled with a search
7. Write findings to Sanctum/00 Inbox/ for your review
```

Lint is what keeps the wiki healthy as it grows.

---

## 17. Phased Build Sequence

Eight phases. The forcing question at every boundary: *"Can I use what I just built to build the next thing?"*

---

### Phase 0 — Substrate

**Status:** Not yet started

Create the repos. Scaffold the structures. Prove the manual loop closes once.

**Builds:**
- Create `sanctum/` git repo (private)
- Create `bifrost/` git repo (public)
- Scaffold Sanctum folder structure per spec
- Scaffold Bifrost folder structure per spec
- Create canonical first docs in each (README, AGENTS.md, system-spec.md)
- Create `Sanctum/01 Indexes/log.md` (empty but present)
- Create `Sanctum/99 Templates/` with all five templates
- Create `Sanctum/30 Memory/Preferences/writing-preferences.md`
- Create `Sanctum/30 Memory/Environment/tooling-and-workflow.md`
- Install Ollama on Mac mini — first model running
- Pi on Mac mini — manual dispatch proven once
- Symlink `bifrost/skills/` to `~/.agents/skills/bifrost/`

**Dogfood:** vault-sync commits the scaffold to GitHub. Manual Pi dispatch proves the shell works.

---

### Phase 1 — GitHub Read Skills

**Status:** Not yet started

Agents can now observe repositories. No side effects.

**Builds:**
- `github/read-file` SKILL.md
- `github/read-diff` SKILL.md
- `github/read-history` SKILL.md
- `github/read-pr` SKILL.md

**Dogfood:** Skills are committed to Bifrost. Agents can now read the commit that added them.

---

### Phase 2 — First Skill + Loop-back

**Status:** Not yet started

First working producer. Prove the quality loop closes. This is the most important phase.

**Builds:**
- `docs/changelog` SKILL.md — reads diff, writes changelog
- `docs/drift` SKILL.md — checks code/docs alignment
- Loop-back wrapper (`bifrost/scripts/loop-back.sh` — ~50 lines)
- Manual Inbox workflow — output reviewed before any PR
- Promote gate — manual review and merge

**Dogfood:** `docs/changelog` writes its own changelog entry after being built. `docs/drift` checks its own SKILL.md for drift from its declared behaviour. The loop-back runs on both.

---

### Phase 3 — Wiki Core Loop

**Status:** Not yet started

Sanctum becomes a real compiling wiki, not just a folder structure.

**Builds:**
- `wiki/ingest` SKILL.md — source → wiki pages + index + log
- `wiki/query` SKILL.md — question → answer + filed wiki page
- `wiki/lint` SKILL.md — wiki health check → Inbox report
- Ingest the system's own design documents as first sources
- Run first lint pass

**Dogfood:** `wiki/ingest` files the notes from its own creation session into Sanctum. `wiki/query` answers "what is the promote gate?" and files the answer as a wiki page. The system begins compounding.

---

### Phase 4 — GitHub Write Skills

**Status:** Not yet started

Agents materialise output as PRs. Review shifts from Inbox files to GitHub PRs.

**Builds:**
- `github/create-branch` SKILL.md
- `github/commit-file` SKILL.md
- `github/open-pr` SKILL.md (always draft; loop-back must pass first)
- `docs/changelog` now opens PRs instead of Inbox files

**Dogfood:** Write skills tested by having `docs/changelog` open its first real PR — on the changelog for adding the write skills themselves.

---

### Phase 5 — Slack + Tachyon Gateway

**Status:** Not yet started

Dispatch from anywhere. The system becomes ambient.

**Builds:**
- Tachyon: Ollama running (1-3B ARM models)
- Tachyon: Pi installed and tested
- Slack workspace — channel structure per taxonomy
- Tachyon: webhook listener wired to Slack
- Tachyon → Mac mini routing for heavy tasks

**Dogfood:** Dispatch `docs/changelog` from phone via Slack. It runs on Tachyon, opens a PR, posts the link back to Slack. First fully ambient loop.

---

### Phase 6 — Self-Improvement Loop

**Status:** Not yet started

The promote gate gets tooling. Memory updates become PRs.

**Builds:**
- `memory/write` skill (session-promoter)
- `github/comment-pr`, `github/request-review`, `github/mark-ready` SKILL.md
- `index/rebuild` skill
- Backfill: Phases 0-5 session summaries ingested into Sanctum
- harness-feedback review workflow established

**Dogfood:** `memory/write` promotes its own build session into a memory note. The system stores a record of building itself.

---

### Phase 7 — Full Workflow Chains

**Status:** Not yet started

Full workflow chains live. Model routing rules active. Real product work on FODMAPP.

**Builds:**
- `react-feature` workflow
- `api-endpoint` workflow
- `rag-pipeline` workflow
- `type-safety` workflow
- OSS model routing rules — mini vs Tachyon by task class

**Dogfood:** `react-feature` workflow builds a Slack channel component for the system's own dispatch interface. System builds its own front door.

---

### Phase 8 — Webhook Triggers

**Status:** Not yet started

Pull becomes push for proven roles. Qualification: ≥2 weeks predictable pull-mode.

**Builds:**
- `github/on-ci-failure` — dispatches bug-find on CI fail
- `github/on-pr-open` — dispatches review workflow on PR open
- `github/on-push-main` — dispatches changelog and drift on push to main

**Dogfood:** CI failure on the system's own repo triggers a bug-find action. The system debugs itself.

---

## 18. Conventions

| Convention | Format | Examples |
|---|---|---|
| Skill IDs | `domain/action` kebab-case | `react/implement` · `docs/changelog` · `wiki/ingest` |
| Workflow IDs | `domain-lifecycle` | `react-feature` · `api-endpoint` · `wiki-maintain` |
| Branch — human | `feat/*` · `fix/*` · `chore/*` · `docs/*` | `feat/ingredient-search` |
| Branch — agent | `bg/*` · `codex/*` · `skill/*` | `bg/ingredient-search` · `skill/react-implement` |
| Tmux session names | `[role]:[task-slug]` | `react-worker:ingredient-search` |
| Session summaries | `[project]-[YYYY-MM-DD].md` | `fodmapp-2026-04-20.md` |
| Inbox files | `[skill-id]-[date].md` | `react-review-2026-04-20.md` |
| Gate failure files | `gate-failure-[branch].md` | `gate-failure-bg-ingredient-search.md` |
| Log entries | `## [YYYY-MM-DD] op \| detail` | `## [2026-04-20] ingest \| Karpathy LLM Wiki` |
| Commit format | Conventional Commits | `feat(ui): add ingredient search components` |
| Memory confidence | `verified` · `inferred` | `confidence: inferred` |
| Note status | `evergreen` · `working` · `archived` | `status: working` |

---

## 19. Existing Infrastructure

FODMAPP has significant infrastructure that Phase 0 builds on. The system is not starting from scratch — but everything system-level will be built fresh in the new repos.

### What exists in FODMAPP and carries forward

- `AGENTS.md` — full domain contract. Template for all future project AGENTS.md files.
- `eslint.llm.config.mjs` — the LLM lint surface. Loop-back's primary signal source.
- `.github/scripts/quality-gate.sh` — fast and `--full` modes. Already production-grade.
- `.githooks/pre-push` — automatic `--full` gate on every push.
- `codex/*` branch prefix — agent branches already distinguishable.
- `skills/turborepo-*/SKILL.md` — three real skill files. Prior art for skill format.
- Worktree model — maps onto foreground/background tier split.
- Documentation governance — PR template, lifecycle vocabulary, persona docs.

### What Phase 0 must create fresh

1. `sanctum/` repo with full folder structure and first canonical docs
2. `bifrost/` repo with full folder structure and contracts
3. `sanctum/01 Indexes/log.md` — empty but present from day one
4. Symlink `bifrost/skills/` to `~/.agents/skills/bifrost/`
5. Ollama on Mac mini + Pi install + manual dispatch test

### The loop-back wrapper is the highest-priority new artifact

Everything needed already exists: `quality-gate.sh` produces exit codes, `eslint.llm.config.mjs` produces structured JSON violations, Pi can re-invoke the agent with a prompt. The wrapper is the connective tissue — ~50 lines. Build it in Phase 2 immediately after the first skill is proven.

---

*End of system-spec v0.2*
