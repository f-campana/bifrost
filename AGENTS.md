# AGENTS.md

## Identity
Bifrost is the schema repo. It defines how agents operate across the system.

## Authorised actions
- Agents may create and edit contracts, skills, scripts, and other schema files.
- Agents may not add source knowledge that belongs in Sanctum.
- Agents may not merge Sanctum and Bifrost concerns.
- Keep design skills generic; the Rune carries the taste.

## Quality gate
Phase 0 uses human review plus `git diff --check`.

## Skill loading
Load skills from `skills/design/_index.md` when working on the design domain.

## Context boot
Read in this order:
1. `system-spec.md`
2. `README.md`
3. `contracts/design-pipeline-contract.md`
4. `skills/design/_index.md`
5. the relevant `skills/design/*/SKILL.md`

## Feedback channel
Write repo feedback to `../sanctum/00 Inbox/harness-feedback.md`.

