---
name: design/validator
version: 1.0.0
type: validator
domain: design
tier: action
triggers:
  - /design-validator
  - "validate design output"
reads_from: DESIGN_[PROJECT].md
writes_to: validation report
model_preference: claude-sonnet
---

# design/validator

## When to use
Use after `design-builder` has produced HTML and the output needs to be checked
against the Design Contract.

## What it does
1. Reads the HTML and the Design Contract.
2. Checks the output against the Rune's inviolable rules.
3. Writes a validation report with findings only.

## What it does NOT do
- It does not edit the HTML.
- It does not mutate the Design Contract.
- It does not approve by default.

## Inputs
- Required: HTML output and Design Contract.
- Optional: any builder run report that helps explain the implementation.

## Output
Validation report with pass/fail findings and concrete violations.

## Failure modes
- Missing contract: stop and report that validation cannot proceed.
- Missing HTML: stop and report the missing artifact.
- Mismatch with the Rune: enumerate each violation separately.

