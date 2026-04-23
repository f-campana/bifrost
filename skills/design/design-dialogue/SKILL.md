---
name: design/dialogue
version: 1.0.0
type: producer
domain: design
tier: action
triggers:
  - /design-dialogue
  - "explain this rune"
reads_from: Sanctum/10 Knowledge/Design/runes/[slug].md
writes_to: answer text
model_preference: claude-sonnet
---

# design/dialogue

## When to use
Use when someone asks for an explanation of a Rune, a Design Contract, or the
reasoning behind a design rule.

## What it does
1. Reads the relevant Rune or contract.
2. Explains the rule set in plain language.
3. Keeps the answer grounded in the system's vocabulary.

## What it does NOT do
- It does not select a Rune.
- It does not write design files.
- It does not expand the domain scope.

## Inputs
- Required: one Rune or Design Contract, plus a question.
- Optional: a specific section or rule to explain.

## Output
Clear explanatory text for the human.

## Failure modes
- Vague question: narrow it before answering.
- Missing context: ask for the Rune or contract first.
- Scope creep: answer only the requested design question.

