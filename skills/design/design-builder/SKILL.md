---
name: design/builder
version: 1.0.0
type: producer
domain: design
tier: action
triggers:
  - /design-builder
  - "build from DESIGN_"
reads_from: DESIGN_[PROJECT].md
writes_to: build output + run report
model_preference: claude-sonnet
---

# design/builder

## When to use
Use when a Design Contract already exists and the task is to turn it into a
first-pass HTML implementation plus a run report.

## What it does
1. Reads the Design Contract only.
2. Builds the interface according to the contract.
3. Writes the HTML output.
4. Writes a run report that records the Rune, contract version, and key checks.

## What it does NOT do
- It does not select the Rune.
- It does not rewrite the Design Contract.
- It does not pull in extra knowledge from Sanctum sources.

## Inputs
- Required: one Design Contract.
- Optional: project assets explicitly named in the contract.

## Output
HTML implementation and a run report in the agreed project output path.

## Failure modes
- Missing or incomplete contract: stop and report the missing fields.
- Conflict with inviolable rules: suppress the violation, do not ship it.
- Generic fallback behaviour: treat as a failure and surface it in the run report.

