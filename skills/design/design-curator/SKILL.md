---
name: design/curator
version: 1.0.0
type: producer
domain: design
tier: action
triggers:
  - /design-curator
  - "instance a design contract"
reads_from: Sanctum/10 Knowledge/Design/runes/[slug].md
writes_to: DESIGN_[PROJECT].md
model_preference: claude-sonnet
---

# design/curator

## When to use
Use when a human has already selected a Rune and the task is to instantiate that
Rune into a project-specific Design Contract.

## What it does
1. Reads the selected Rune and the brief.
2. Resolves project naming and contract metadata.
3. Writes `DESIGN_[PROJECT].md` at the project root.
4. Keeps the contract self-contained and ready for `design-builder`.

## What it does NOT do
- It does not select the Rune.
- It does not build HTML.
- It does not invent new design identity rules.

## Inputs
- Required: selected Rune file, brief, project name.
- Optional: target path, additional project constraints already approved by the
  human.

## Output
One Design Contract in Google `design.md` format at the project root.

## Failure modes
- Missing Rune or brief: stop and request the missing input.
- Conflicting project naming: stop and preserve the human's chosen name.
- Contract drift: prefer the Rune and brief over generic defaults.

