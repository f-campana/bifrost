---
name: design/harvester
version: 1.0.0
type: producer
domain: design
tier: action
triggers:
  - /design-harvester
  - "harvest design runs"
reads_from: run reports
writes_to: harvest report
model_preference: claude-sonnet
---

# design/harvester

## When to use
Use when one or more design runs have completed and their run reports need to
be summarized into reusable findings.

## What it does
1. Reads run reports.
2. Groups repeated patterns, failures, and useful constraints.
3. Writes a harvest report for human review.

## What it does NOT do
- It does not change the original run reports.
- It does not promote findings into Sanctum on its own.
- It does not invent new Rune identities.

## Inputs
- Required: one or more run reports.
- Optional: validation reports that clarify recurring issues.

## Output
Harvest report with repeatable observations and candidate improvements.

## Failure modes
- No run reports: stop and report that there is nothing to harvest.
- Inconsistent report format: normalize what you can and flag the rest.
- Overreach: keep the output as evidence, not redesign.

