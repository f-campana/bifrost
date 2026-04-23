# Hybrid Agent System — Full Specification

**Version:** 0.1-draft  
**Status:** In progress  
**Primary repo:** f-campana/fodmapp  
**Vault:** fabien-knowledge-os  
**Last updated:** 2026-04-19

---

## Table of Contents

1. [Overview](#1-overview)
2. [Core Principles](#2-core-principles)
3. [Build Status](#3-build-status)
4. [The Vault](#4-the-vault)
5. [Agent Taxonomy](#5-agent-taxonomy)
6. [Hardware Topology](#6-hardware-topology)
7. [Model Routing](#7-model-routing)
8. [Interface Layer](#8-interface-layer)
9. [AGENTS.md Contract](#9-agentsmd-contract)
10. [SKILL.md Contract](#10-skillmd-contract)
11. [GitHub Skills](#11-github-skills)
12. [Quality Gate](#12-quality-gate)
13. [Loop-back Gate](#13-loop-back-gate)
14. [Promote Gate + Self-Improvement Loop](#14-promote-gate--self-improvement-loop)
15. [Phased Build Sequence](#15-phased-build-sequence)
16. [Conventions](#16-conventions)
17. [Existing Infrastructure](#17-existing-infrastructure)

---

## 1. Overview

This document specifies a hybrid human-orchestrated agent system for software development. It defines the architecture, contracts, roles, hardware topology, build sequence, and conventions required to implement and operate the system.

The system is built incrementally — each phase uses what was built in the previous phase to build what comes next. It is designed to be agnostic across harnesses (Claude Code, Codex, Pi) and uses Markdown files as its primary communication protocol. The orchestrator is a human. Automation is earned, not assumed.

The primary product being built with and on this system is **FODMAPP** — an open FODMAP tracking and planning platform. The system is simultaneously the development tool and the subject of its own development (dogfooding).

### What this system is not

This is not an automated orchestration framework. There is no AI orchestrator. There are no agents spawning other agents without human dispatch. The system is a set of well-defined, scoped agents that a human routes between. The intelligence lives at the top of the stack, not in a routing layer.

---

## 2. Core Principles

### Human orchestrator
The human makes all routing decisions, task decompositions, and promote gate judgments. No agent decides which agent runs next in production. Automation is for execution, not for orchestration.

### Vault as bus
All agent communication happens through Markdown files in the vault. Files are the API. No custom message-passing, no shared runtime state, no proprietary coordination layer. The vault is version-controlled and human-readable at all times.

### Harness agnosticism
`AGENTS.md` and `SKILL.md` are the harness contract. Pi, Claude Code, and Codex all read them. The system does not depend on any single harness. Skills written once work across all harnesses that implement the agentskills.io standard.

### Pull before push
Every automated trigger starts as a manual dispatch. A role earns automated triggering only after it has been dispatched manually and predictably for at least two weeks. Automation is a reward for proven reliability, not a starting assumption.

### Dogfood the build
Each phase uses what was built in the previous phase to build what comes next. The system builds itself. Agents write the changelog for the agents. Skills document the skills. The quality gate validates the quality gate scripts.

### Full observability
Every gate check is a command you can run yourself. Every agent output is a file you can read. Every failure is structured output you can inspect. Nothing the agent sees is hidden from you.

### Promote gate
Nothing moves from agent output to durable memory without human review. Background agents write to `00 Inbox/` only. You review and approve before anything is promoted to `30 Memory/` or committed to the main branch.

### Narrow context
Agents load only what they need. Action agents operate on ≤10 files. Workflow agents load one domain map. Project agents load cross-domain maps. Strategy agents (you) hold the full vault. Loading more context than needed causes scope creep and costs more — in both tokens and correctness.

---

## 3. Build Status

| Capability | Status | Location |
|---|---|---|
| Root AGENTS.md | ✅ built | `AGENTS.md` — full domain contract for FODMAPP |
| LLM lint surface | ✅ built | `eslint.llm.config.mjs` — warnings as errors for agent code |
| CLI quality gate | ✅ built | `.github/scripts/quality-gate.sh` — fast and `--full` modes |
| Pre-push enforcement | ✅ built | `.githooks/pre-push` — runs `--full` gate on every git push |
| Agent branch identity | ✅ built | `codex/*` branch prefix — agent branches distinguishable in git |
| Skills folder | 🟡 partial | `skills/` — exists, `$repo-git-hygiene-bootstrap` confirmed, needs taxonomy mapping |
| Worktree model | ✅ built | `docs/ops/worktree-status.md` — one initiative per worktree |
| `50 Sessions/` in vault | ❌ not yet | Phase 0 — add to vault, add session summary instruction to AGENTS.md |
| Loop-back wrapper | ❌ not yet | Phase 2.5 — ~50 lines, feeds gate failure JSON back to agent |
| GitHub read skills | 🟡 partial | gh CLI present, not yet formalised as SKILL.md files |
| GitHub write skills | ❌ not yet | Phase 3 — branch, commit, open-pr as SKILL.md |
| Slack gateway | ❌ not yet | Phase 4 — Tachyon webhook listener + Slack workspace |
| Ollama on Mac mini | ❌ not yet | Phase 0 — prerequisite for background tier |
| Tachyon configured | ❌ not yet | Phase 4 — Pi, Ollama 1-3B, webhook listener, SSH from mini |
| Session-promoter agent | ❌ not yet | Phase 5 — reads flagged sessions, drafts memory note updates as PRs |

---

## 4. The Vault

The vault (`fabien-knowledge-os`) is the system's shared state and communication bus. It is a git-backed Obsidian vault synced via `scripts/vault-sync`. All agents read from and write to the vault via the Markdown protocol below.

### Folder access contract

| Path | Read | Write | Rule |
|---|---|---|---|
| `00 Inbox/` | you | background agents | All background output lands here first. Nothing leaves without your review. `harness-feedback.md` lives here — agents append improvement suggestions. |
| `10 Knowledge/` | foreground agents | self-improve agents (via PR) | Lessons, concepts, exercises. Updated only via the promote gate. Never written directly by background agents. |
| `20 Projects/` | foreground agents | foreground agents | Active project context. Foreground reads and writes freely. Background agents read task specs from here. |
| `30 Memory/` | foreground agents | you only | Durable preferences, decisions, and constraints. Gate-protected. Background agents never write here. |
| `40 Sources/` | foreground agents | source agent | Raw imported material. Foreground reads for external context. |
| `50 Sessions/` | background agents | foreground agents | **New — not yet created.** Session summaries with `promote_to_memory` flag. The handoff zone. |
| `99 Templates/` | all agents | you only | Note schemas. Agents reference for output format. Not modified by agents. |
| `index.md` → maps | foreground entry point | index-rebuilder | Navigation root. Always loaded first by foreground agents. Maps are the context ceiling for workflow agents. |

### Note templates

All agents producing vault notes must use the appropriate template from `99 Templates/`.

| Template | Frontmatter | Sections |
|---|---|---|
| `concept-note.md` | type, status (evergreen), updated, tags | Summary, Why it matters, Related |
| `memory-note.md` | type, scope, confidence (verified\|inferred), updated | Verified, Inferred |
| `project-note.md` | type, project, status, updated | Context, Decisions, Constraints, Related |
| `session-summary.md` | type, agent, project, date, promote_to_memory (no\|yes) | Objective, What happened, Decisions, Follow-up |
| `source-note.md` | type, source_kind, author, captured | Summary, Relevance, Extracted ideas |

---

## 5. Agent Taxonomy

Four tiers. The distinguishing property between tiers is **scope of judgment and context budget** — not complexity. Domain verticals (React, TypeScript, RAG, ML, Architecture, Design, Security, Docs) cut through all tiers.

---

### Tier 1 — Actions

> One domain × one verb · ≤10 files · deterministic output · background tier

Action agents have two subtypes:

- **Producers** — generate an artifact. Never validate.
- **Validators** — inspect an artifact produced by another agent. Never fix. Report only.

This separation is strict.

#### Producer actions

| ID | Input | Output |
|---|---|---|
| `react:implement` | component spec | component file(s) |
| `react:test` | component file | test suite |
| `react:story` | component file | Storybook stories |
| `react:migrate` | class component | functional component |
| `ts:type` | shape / runtime value | type definitions |
| `ts:narrow` | unsafe branches | narrowed branches |
| `ts:infer` | runtime values | extracted types |
| `express:route` | route spec | handler file |
| `express:middleware` | requirement | middleware file |
| `express:schema` | payload shape | Zod/Joi schema |
| `docs:write` | code or spec | documentation file |
| `docs:changelog` | git diff (via `gh:read-diff`) | changelog entry → PR |
| `docs:adr` | decision brief | Architecture Decision Record |
| `rag:chunk` | raw documents | chunked documents |
| `rag:embed` | chunked documents | embeddings stored |
| `rag:retrieve` | query + vector store | retrieval interface |
| `data:transform` | source data | transformed data |
| `ml:feature` | raw dataset | feature-engineered dataset |
| `ml:eval` | model + test set | evaluation harness |
| `nlp:extract` | text | entities / intents |
| `design:token` | design brief | token files (DTCG) |
| `design:spec` | brief | component spec |
| `design:a11y-fix` | component markup | accessibility-patched markup |
| `session-promoter` | session summary (promote_to_memory: yes) | draft memory note update → Inbox PR |
| `index-rebuilder` | vault folder scan | updated map files + dead-link report |

#### Validator actions

| ID | Input | Output |
|---|---|---|
| `react:review` | diff or PR | review report → Inbox |
| `react:bug-find` | failing file(s) | bug report → Inbox |
| `react:perf-audit` | component + render profile | perf report → Inbox |
| `ts:audit` | module files | type safety report → Inbox |
| `ts:strict` | module files | strict mode compliance report → Inbox |
| `arch:solid` | module files | SOLID audit → Inbox |
| `arch:clean` | module files + arch spec | layer compliance report → Inbox |
| `arch:ddd-review` | domain model files | DDD audit → Inbox |
| `arch:event-review` | event model files | event sourcing audit → Inbox |
| `sec:audit` | code surface (diff or files) | vulnerability report → Inbox |
| `sec:auth-review` | auth/middleware files | auth model audit → Inbox |
| `sec:injection` | route/handler files | injection surface report → Inbox |
| `rag:quality` | retrieval results + ground truth | quality/recall report → Inbox |
| `data:schema-validate` | data + schema | compliance report → Inbox |
| `ml:bias-check` | model outputs | bias detection report → Inbox |
| `ml:leakage` | pipeline files | data leakage report → Inbox |
| `design:token-validate` | token files | parity/consistency report → Inbox |
| `design:contrast` | component markup | WCAG contrast report → Inbox |
| `design:a11y-audit` | component markup | full a11y compliance report → Inbox |
| `docs:drift` | code files + docs | drift report → Inbox |
| `docs:coverage` | public API surface | undocumented surface report → Inbox |

---

### Tier 2 — Workflows

> Single domain · full lifecycle · chains actions · background (routine) or foreground (complex)

A Workflow chains producers and validators toward a complete domain outcome. It requires domain judgment about action ordering and edge cases. BDD is a workflow protocol — it changes the ordering of the same actions, not the actions themselves.

Validators in chains are marked with `⟨⟩`.

| ID | Domain | Chain | Tier |
|---|---|---|---|
| `react-feature` | React | implement → test → story → ⟨review⟩ → ⟨perf-audit⟩ → docs:write | background (routine) |
| `api-endpoint` | Express | route → schema → ⟨auth-review⟩ → ⟨sec:injection⟩ → docs:write | background (routine) |
| `type-safety` | TypeScript | ts:type → ts:narrow → ⟨ts:audit⟩ → ⟨ts:strict⟩ | background (routine) |
| `rag-pipeline` | RAG / Data | chunk → embed → retrieve → ⟨rag:quality⟩ → ⟨data:schema-validate⟩ | background (routine) |
| `design-system` | Design | token → spec → ⟨token-validate⟩ → ⟨contrast⟩ → ⟨a11y-audit⟩ | background (routine) |
| `ddd-bounded-context` | Architecture/DDD | docs:adr → schema → ⟨arch:ddd-review⟩ → ⟨arch:solid⟩ → docs:write | foreground (complex) |
| `bdd-cycle` | BDD (ordering protocol) | docs:write(spec) → react:test(stubs) → react:implement → ⟨react:review⟩ | foreground (complex) |
| `ml-experiment` | ML | ml:feature → ml:eval → ⟨ml:bias-check⟩ → ⟨ml:leakage⟩ → docs:write | foreground (complex — different lifecycle) |

---

### Tier 3 — Projects

> Multiple domains · single goal · coordinates workflows · foreground tier

Projects coordinate workflows across domain boundaries. You are present and steering. You route between workflows, resolve conflicts, and make tradeoff calls.

| ID | Workflows | Notes |
|---|---|---|
| `full-stack-feature` | react-feature + api-endpoint + type-safety | + sec:audit + docs:changelog on completion |
| `event-sourced-module` | ddd-bounded-context + api-endpoint | arch:event-review + arch:clean injected at project level |
| `search-product` | rag-pipeline + react-feature | + nlp:extract + react:perf-audit on UI |
| `design-system-rebuild` | design-system + react-feature | + docs:coverage on all public components |
| `ml-feature-to-prod` | ml-experiment + api-endpoint | Mandatory human checkpoint between ML eval and API wiring |

---

### Tier 4 — Strategy

> Product / roadmap scope · full vault · you are the primary agent

Strategy requires the full vault, all project context, memory, and external research. You are the Strategy agent. Automated Strategy agents require your explicit approval before any downstream action.

| ID | Description |
|---|---|
| `architecture-decision` | Evaluates architectural options across the full system. Invokes arch validators across all candidates. Produces ADR. |
| `product-ideation` | Broad exploration. No fixed agent chain. You orchestrate directly. Produces decision memo or opportunity map. |
| `codebase-evolution` | Multi-sprint refactor plan. Decomposes into sequenced Projects. Identifies ordering constraints and blast radius. |
| `system-audit` | Full validator sweep across the codebase. Produces prioritised remediation backlog. Closest to an automated Strategy agent — still requires your approval before remediation runs. |

---

## 6. Hardware Topology

### MacBook Pro
**Role:** Foreground tier

| Property | Value |
|---|---|
| Tools | Claude Code, Codex App, Ghostty, Tmux |
| Models | Claude Sonnet / Opus, Codex (OpenAI) |
| Agent tiers | Strategy, Projects, Complex Workflows |
| Vault | Primary read/write, vault-sync → GitHub |

### Mac mini M1
**Role:** Background tier — heavy workers

| Property | Value |
|---|---|
| Tools | Pi workers, Tmux named sessions, Ollama, GitHub CLI |
| Models | Qwen2.5-Coder-32B, Mistral-Small, Claude API (fallback) |
| Agent tiers | Routine Workflows, Heavy Validators, Mechanical Actions |
| Connectivity | Local network only — SSH accessible |
| Vault | Read-only clone, PR writes via GitHub CLI |

### Particle Tachyon
**Role:** Background tier — gateway + lightweight workers

| Property | Value |
|---|---|
| Hardware | Qualcomm QCM6490, 8GB RAM, 128GB, Ubuntu 24.04 |
| Tools | Pi workers, Slack webhook listener, Ollama (ARM), GitHub CLI |
| Models | Phi-3-mini, Qwen2.5-Coder-1.5B, Qualcomm AI Hub (future) |
| Agent tiers | Dispatch router, Lightweight Actions, Watcher agents, Webhook triggers |
| Connectivity | 5G EtherSIM (independent of local network), WiFi 6E |
| Vault | Read-only clone, PR writes via GitHub CLI |

### Key property: the Tachyon is the gateway

The Mac mini is only reachable on the local network. The Tachyon has an independent 5G connection. It receives all Slack dispatch messages and GitHub webhooks regardless of local network state, routes heavyweight tasks to the Mac mini over the local network, and handles lightweight actions locally. It is the always-on ambient interface layer.

---

## 7. Model Routing

> Start with everything on Claude. Migrate to OSS models as each node comes online. Never guess about model quality — migrate with evidence from real Claude runs.

| Task class | Model | Node | Rationale |
|---|---|---|---|
| Strategy · Planning | Claude Opus / Sonnet | MBP | Maximum reasoning. Output shapes irreversible decisions. No substitute. |
| Complex Workflows | Claude Sonnet | MBP | Cross-action judgment required. OSS models risk wrong ordering. |
| Routine Workflows | Ollama 7B+ (Qwen2.5-Coder-32B) | Mac mini | Deterministic chains. OSS performs well when ordering is specified in AGENTS.md. |
| High-stakes Validators | Claude Sonnet | MBP | Findings may change architectural direction. |
| Routine Validators | Ollama 7B+ | Mac mini | Pattern matching. Deterministic checks. |
| Mechanical Producers | Ollama 1–3B (Phi-3-mini) | Tachyon | Narrow input spec, single file output. Fast and sufficient. |
| Dispatch + routing | No model (Node.js) | Tachyon | Pure control plane logic. LLM involvement is wasteful. |

**Routing escalation rule:** If an OSS model's output consistently fails your review or the loop-back gate, escalate that task class to Claude. Write the exception into the AGENTS.md routing section. The feedback-merger picks it up.

**Migration sequence:** P0–P3: all Claude → P4: mini handles routine → P5+: Tachyon handles mechanical → P7: full matrix active.

---

## 8. Interface Layer

### Foreground — Claude Code and Codex Apps (MBP)

Active pairing sessions. High-attention, conversational, back-and-forth judgment. Architecture decisions, complex debugging, design decisions. Sessions always end with a session summary written to `50 Sessions/` with a `promote_to_memory` gate.

### Background — Ghostty + Tmux named sessions (Mac mini + Tachyon)

Named tmux sessions map to named agent roles. Each boots with a scoped AGENTS.md and narrow context. You dispatch and check back.

Session naming convention: `[role]:[task-slug]` — e.g. `react-worker:ingredient-search`

### Ambient — Slack (Tachyon gateway)

A dedicated Slack workspace receives dispatch from any device (phone, MBP, any network) and posts agent output, PR links, and Inbox review requests back as threads.

```
#dispatch          ← you post tasks here from any device
#inbox-review      ← all agent outputs land here for your approval
#react-worker      ← react-feature workflow channel
#api-worker        ← api-endpoint workflow channel
#docs-worker       ← changelog, drift detection, docs:write
#rag-worker        ← rag-pipeline workflow channel
#maintenance       ← stale scanner, index-rebuilder, feedback-merger
#harness-feedback  ← agents post AGENTS.md improvement suggestions
```

### Trigger model

**Pull now.** You dispatch manually. Every role starts in pull mode.

**Push later (Phase 7).** Roles dispatched predictably for ≥2 weeks in pull mode become candidates for automated triggers. Qualification criterion: your judgment adds nothing to the trigger decision. Only then automate it. The gate still holds.

---

## 9. AGENTS.md Contract

Every agent context boots from an AGENTS.md file. Root AGENTS.md covers the full repository. Scoped AGENTS.md files are per-role and per-context. Both are version-controlled and reviewed like code.

**Size constraint:** Root AGENTS.md: under 200 lines. Scoped AGENTS.md: under 80 lines. If a file exceeds these limits, it contains too much — split into a referenced file.

### Root AGENTS.md template (foreground tier)

```markdown
# AGENTS.md

## Scope
Apply to the whole repository root.

## Context Boot Sequence
1. Read index.md from the vault to understand navigation structure.
2. Load the domain map that matches the task (react-map, backend-map, typescript-map, etc.).
3. Load the relevant project note from 20 Projects/ if the task is project-scoped.
4. Load applicable memory notes from 30 Memory/ that match the task domain.
5. Do not load lessons or full concept files unless the task requires that depth.

## Skill Routing
[List skill triggers and their SKILL.md paths]

## Collaboration Contract
- Short-lived branches from main.
- Conventional Commit format.
- Semantic PR titles.
- Run quality-gate.sh before requesting merge.
- Loop-back gate runs before opening any PR.

## Safety Rules
- Never rewrite history on shared branches.
- Never commit secrets or credentials.
- Never write directly to 30 Memory/ — Inbox only.

## Session Summary Instruction
At the end of every session, write a session summary to 50 Sessions/ using the
session-summary template. Set promote_to_memory: no by default. Set to yes only
if the session revealed a decision, constraint, or gap that should persist.
If AGENTS.md itself has a gap or error, append a one-line note to
00 Inbox/harness-feedback.md.

## Reporting
- State changed files and why.
- State validation commands run and results.
- State remaining risk or follow-up.
```

### Scoped AGENTS.md template (background tier — per role)

```markdown
# AGENTS.md — [role-id]

## Role
[One sentence: what this agent does, nothing else.]

## Scope
[Exact paths this agent may read. Exact output path.]

## Skill
[Path to the SKILL.md file for this role.]

## Gate
[Which quality gate script runs on output before PR is opened.]
[max_iterations: 3]
[escalate_on_failure: foreground]

## Input Contract
[What this agent receives as input. Structured precisely.]

## Output Contract
[What this agent produces. Where it writes. Format required.]

## Stop Condition
[When the agent stops and waits. Never self-escalates beyond max_iterations.]

## Forbidden
[What this agent must never do. At minimum: never write to 30 Memory/.]
```

---

## 10. SKILL.md Contract

A skill file packages a reusable capability. Skills conform to the agentskills.io standard and are portable across Pi, Claude Code, and Codex. Skills live in `skills/` at the repo root.

```markdown
---
name: [domain]:[verb]
version: 1.0.0
type: producer | validator
domain: react
tier: action | workflow
triggers:
  - /react-implement
  - "implement this component"
gates:
  - .github/scripts/quality-gate.sh --llm-only
max_iterations: 3
escalate_on_failure: foreground
model_preference: ollama-7b
node_preference: mini
---

# [Skill name]

## When to use this skill
[Precise trigger conditions. One scenario, not many.]

## What it does
[What the skill produces or validates. Concrete, not abstract.]

## Input
[Exact expected input format and required files.]

## Steps
1. [First step — specific and executable]
2. [Second step]
...

## Output
[Exact output format, target path, and how it is written.]

## Failure modes
[What goes wrong and what the agent should do on each failure.]

## Examples
[At least one concrete input → output example. Anti-examples if helpful.]
```

**Key property — injectable validators:** Validator skills have no fixed position in a workflow. They slot in after any producer agent. They never fix what they find — they report it. A workflow AGENTS.md specifies where validators are injected in the chain.

---

## 11. GitHub Skills

Four capability tiers. Each tier unlocks a new class of agent behaviour. Read skills must exist before write skills. Write skills must exist before PR workflow skills.

| Tier | Phase | Gate | Skills |
|---|---|---|---|
| **Read** | Phase 1 | None — read-only, no side effects | `gh:read-file` · `gh:read-diff` · `gh:read-history` · `gh:read-pr` |
| **Write** | Phase 3 | You review PR before merge. Always branch + draft PR. Never main directly. | `gh:create-branch` · `gh:commit-file` · `gh:open-pr` |
| **PR Workflow** | Phase 5 | You approve merge. Agents participate in review lifecycle. | `gh:comment-pr` · `gh:request-review` · `gh:mark-ready` |
| **Webhooks** | Phase 7 | ≥2 weeks pull-mode proven. Gate still holds. | `gh:on-ci-failure` · `gh:on-pr-open` · `gh:on-push-main` |

**Write skill invariant:** A write skill may only be invoked after the agent's output has passed the loop-back gate. `gh:open-pr` is never the first thing an agent calls. If the gate fails and max iterations are exhausted, the agent escalates to foreground rather than opening a PR.

---

## 12. Quality Gate

`.github/scripts/quality-gate.sh` is the system's primary enforcement mechanism. It already exists in FODMAPP. The loop-back wrapper is the missing piece.

### Fast mode (loop-back iterations)

Command: `quality-gate.sh` (or `--llm-only`)

Runs: `pnpm lint:llm` + `tsc --noEmit`

Target: under 30 seconds. Produces structured JSON output. Designed for agent retry loops.

### Full mode (pre-push + CI)

Command: `quality-gate.sh --full`

Runs: `prettier --check` + `pnpm lint:ci` + `pnpm lint:llm` + `tsc --noEmit` + `pnpm test` + `pnpm build` + `changeset:ci:lint:all` + `ci-api-pr.sh` (when applicable).

Already enforced via `.githooks/pre-push` and GitHub Actions CI.

### LLM lint surface

`eslint.llm.config.mjs` applies stricter rules to agent-produced code:

- All warnings become errors
- `@typescript-eslint/no-explicit-any` → error
- `@typescript-eslint/ban-ts-comment` → error
- `react-hooks/exhaustive-deps` → error
- `no-console` → error
- `prefer-const` → error

Human code warnings remain warnings. CI runs `lint:llm:ci` as a hard failure on all `codex/*` branches.

**Key property — full observability:** Every gate check is a command you can run yourself. The agent sees the same output you see when you run `pnpm lint:llm 2>&1 | jq`. The loop-back feeds the agent structured ESLint JSON — the agent never guesses what went wrong.

---

## 13. Loop-back Gate

The loop-back wrapper (~50 lines of shell or Node.js) runs the fast quality gate, captures structured failure output, and feeds it back to the agent as a structured feedback prompt. It limits retries to a configurable maximum (default: 3) and escalates to the foreground tier on exhaustion.

**Everything needed to build the loop-back already exists in FODMAPP.** The wrapper is the only missing piece.

### Flow

```
1. AGENT       Produces files. Commits to codex/* or bg/* branch.
               Does not push or open a PR yet.

2. GATE        Loop-back runs quality-gate.sh --llm-only.
               Captures exit code and ESLint JSON.
               → Exit 0: proceed to PR.
               → Non-zero: build structured feedback object, re-invoke agent.

3. AGENT       Receives structured feedback. Fixes exactly the reported violations.
               Re-runs gate. Bounded at max_iterations (default: 3).
               → On exhaustion: writes failure log to 00 Inbox/gate-failure-[branch].md
                 and sends Slack notification. Does not open a PR.

4. GATE        Exit 0 confirmed. Loop-back invokes gh:open-pr as a draft.
               PR description includes gate iteration count and lint surface used.
               Branch is pushed at this point.

5. CI          quality-gate.sh --full runs on PR open. Source of truth.
               lint:llm:ci as hard failure. Validator agents may comment on the PR.

6. YOU         Judgment gate. All deterministic checks have passed.
               Review for correctness of abstraction, architectural fit, and intent match.
```

### Feedback object structure

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

## 14. Promote Gate + Self-Improvement Loop

The promote gate is the mechanism by which agent session outputs become durable system knowledge. Two checkpoints: your review of the session summary, and your merge of the background agent's drafted update.

### Flow

```
1. FOREGROUND  Writes session summary to 50 Sessions/[project]-[date].md.
               Default: promote_to_memory: no.
               Set to yes only if the session revealed a persistent decision,
               constraint, or knowledge gap.

2. YOU         Review session summary.
               → No: session archived, no further action.
               → Yes: dispatch session-promoter via Slack.

3. BG AGENT    session-promoter reads the flagged summary.
               Identifies target note (concept, memory, or project note).
               Drafts an update. Opens a draft PR against the vault repository.
               Posts PR link to Slack #inbox-review.

4. YOU         Review the PR. Merge if accurate and worth keeping.
               Background agents never commit directly to 30 Memory/.
               All updates arrive as PRs you merge.
```

### Harness feedback loop

If any session reveals a gap, error, or ambiguity in AGENTS.md or a skill file, the agent appends a one-line note to `00 Inbox/harness-feedback.md`. Review this file weekly. The feedback-merger agent drafts AGENTS.md patch PRs from high-confidence items. You merge. The system's instructions improve over time.

---

## 15. Phased Build Sequence

Eight phases. The forcing question at every phase boundary: *"Can I use what I just built to build the next thing?"*

---

### Phase 0 — Substrate

**Status:** Not yet started

Build the minimum viable foundation. No agents yet. Prove the loop closes manually end-to-end once.

**Builds:**
- Add `50 Sessions/` to vault
- Foreground AGENTS.md with session summary instruction
- Scoped AGENTS.md template for background agents
- Ollama on Mac mini — first model running
- Pi on Mac mini — manual dispatch proven
- Manual loop closed once end-to-end

**Dogfood:** vault-sync commits the AGENTS.md files. Manual Pi dispatch tests the loop.

---

### Phase 1 — GitHub Read Skills

**Status:** Not yet started  
**GitHub skills:** Read tier only

Agents can now observe the repository. No side effects. Safe to deploy without a gate. Prerequisite for every validator and most producers.

**Builds:**
- `gh:read-file` SKILL.md
- `gh:read-diff` SKILL.md
- `gh:read-history` SKILL.md
- `gh:read-pr` SKILL.md

**Dogfood:** vault-sync commits the read skill files to GitHub. Agents can now read the commit that added them.

---

### Phase 2 — First Agents + Loop-back

**Status:** Not yet started  
**GitHub skills:** Read tier

First working producer and validator. Observe failure modes manually. Codify as CLI gate. Build the loop-back wrapper.

**Builds:**
- `docs:changelog` — reads diff, writes changelog
- `docs:drift` — checks code/docs alignment
- Inbox workflow — output lands in `00 Inbox/` manually
- Promote gate — manual review and merge
- Loop-back wrapper (~50 lines) — feeds gate JSON back to agent

**Dogfood:** docs:changelog writes its own changelog entry after being built. docs:drift checks its own AGENTS.md for drift.

---

### Phase 3 — GitHub Write Skills

**Status:** Not yet started  
**GitHub skills:** Read + Write tiers

Agents materialise output as PRs. Review shifts from Inbox files to GitHub PRs. Same gate, better tooling.

**Builds:**
- `gh:create-branch` SKILL.md
- `gh:commit-file` SKILL.md
- `gh:open-pr` SKILL.md (always draft; loop-back must pass first)
- docs:changelog now opens PRs instead of Inbox files

**Dogfood:** Write skills tested by having docs:changelog open its first real PR — on the changelog for adding the write skills themselves.

---

### Phase 4 — Slack + Tachyon Gateway

**Status:** Not yet started  
**GitHub skills:** Read + Write tiers

Dispatch from anywhere. PR links posted to Slack. The system becomes ambient.

**Builds:**
- Tachyon: Ollama running (1-3B ARM models)
- Tachyon: Pi installed and tested
- Slack workspace — channel structure per taxonomy
- Tachyon: webhook listener wired to Slack channels
- Tachyon → Mac mini routing for heavy tasks

**Dogfood:** Dispatch docs:changelog from phone via Slack. It runs on Tachyon, opens a PR, posts link back to Slack. First fully ambient loop.

---

### Phase 5 — Self-Improvement Loop

**Status:** Not yet started  
**GitHub skills:** Read + Write + PR Workflow tiers

The promote gate gets tooling. Memory updates become PRs. Backfill Phases 0–4.

**Builds:**
- `session-promoter` agent
- `gh:comment-pr`, `gh:request-review`, `gh:mark-ready` SKILL.md files
- `index-rebuilder` agent
- Backfill: Phases 0-4 session summaries promoted
- `harness-feedback.md` review workflow established

**Dogfood:** session-promoter promotes its own build session. The system stores a record of building itself.

---

### Phase 6 — Full Workflow Chains

**Status:** Not yet started  
**GitHub skills:** Read + Write + PR Workflow tiers

Full workflow chains live. Model routing rules active. The system is doing real product work on FODMAPP.

**Builds:**
- `react-feature` workflow
- `api-endpoint` workflow
- `rag-pipeline` workflow
- `type-safety` workflow
- OSS model routing rules — mini vs Tachyon by task class

**Dogfood:** react-feature workflow builds a Slack channel component for the system's own dispatch interface. System builds its own front door.

---

### Phase 7 — Webhook Triggers

**Status:** Not yet started  
**GitHub skills:** Full stack including Webhooks tier

Pull becomes push for proven roles. Qualification: ≥2 weeks dispatched predictably in pull mode.

**Builds:**
- `gh:on-ci-failure` — dispatches bug-find / ts:audit on CI fail
- `gh:on-pr-open` — dispatches review workflow on PR open
- `gh:on-push-main` — dispatches changelog and drift on push to main
- Qualification audit — which roles have earned push triggering

**Dogfood:** CI failure on the system's own repo triggers a bug-find action. The system debugs itself.

---

## 16. Conventions

| Convention | Format | Examples |
|---|---|---|
| Agent IDs | `[domain]:[verb]` | `react:implement` · `docs:changelog` · `rag:retrieve` |
| Workflow IDs | `[domain]-[lifecycle]` | `react-feature` · `api-endpoint` · `rag-pipeline` |
| Project IDs | `[intent]-[scope]` | `full-stack-feature` · `event-sourced-module` |
| Branch — human | `feat/*` · `fix/*` · `chore/*` · `docs/*` | `feat/ingredient-search` |
| Branch — agent | `codex/*` · `bg/*` · `skill/*` | `codex/ingredient-search` · `skill/react-implement` |
| Tmux session names | `[role]:[task-slug]` | `react-worker:ingredient-search` |
| Session summary filenames | `[project]-[YYYY-MM-DD].md` | `fodmapp-2026-04-19.md` |
| Inbox files | `[agent-id]-[date].md` | `react-review-2026-04-19.md` |
| Gate failure files | `gate-failure-[branch].md` | `gate-failure-codex-ingredient-search.md` |
| Commit format | Conventional Commits | `feat(ui): add ingredient search components` |
| PR title format | Conventional Commits (semantic) | `feat(ui): add ingredient search components` |
| Memory note `scope` | `project` · `domain` · `global` | `scope: project` (FODMAPP-specific) |
| Memory note `confidence` | `verified` · `inferred` | `confidence: inferred` (not yet tested in production) |

---

## 17. Existing Infrastructure

FODMAPP (`github.com/f-campana/fodmapp`) already has significant infrastructure that Phase 0 builds on.

### What is built and production-grade

- **Root AGENTS.md** — full domain contract with scope, skill routing, collaboration contract, safety rules, worktree model, Phase 3 data contract (scoring semantics, snapshot lock, bilingual requirements, rank 2 exclusion, API sort contract).
- **`eslint.llm.config.mjs`** — dedicated LLM lint surface. Warnings as errors for agent-produced code. `lint:llm:ci` as hard failure in CI.
- **`.github/scripts/quality-gate.sh`** — fast and `--full` modes. Covers format, lint (base + LLM), typecheck, tests, build, changeset validation.
- **`.githooks/pre-push`** — automatic `--full` gate on every `git push`. Delete-only pushes skipped.
- **`codex/*` branch prefix** — agent-produced branches are distinguishable in git history and PR list.
- **Worktree model** — `docs/ops/worktree-status.md`, one initiative per worktree. Maps cleanly onto the foreground/background tier split.
- **Documentation governance** — `docs/README.md`, `docs/foundation/documentation-personas.md`, lifecycle status vocabulary, canonical doc routing. PR template includes Documentation Governance block.
- **Changeset workflow** — `.changeset/`, changeset:ci:lint:all, exemption labeling.

### What Phase 0 needs to do

1. Add `50 Sessions/` to vault. Add session summary instruction to root AGENTS.md.
2. Install Ollama on Mac mini. Confirm Pi runs and can dispatch an agent manually.
3. Write first scoped AGENTS.md for the `docs:changelog` role — use the Phase 3 data section as the structural template for domain contracts.
4. Inspect and map `skills/` folder contents against the Action taxonomy. Document what exists and what is missing.

### The loop-back wrapper is the highest-priority addition

Everything needed to build the loop-back already exists:

- `quality-gate.sh` produces exit codes
- `eslint.llm.config.mjs` produces structured JSON violations
- Pi can re-invoke the agent with a prompt

The wrapper is the connective tissue — approximately 50 lines of shell or Node.js. It transforms the gate from a wall into a feedback channel. Build it in Phase 2 immediately after the first two agents are proven.

```bash
# Reproduce what the agent sees on failure
pnpm lint:llm 2>&1 | jq
```

---

*End of specification v0.1-draft*
