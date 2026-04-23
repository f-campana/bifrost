---
name: github/read-file
version: 1.0.0
type: producer
domain: github
tier: action
triggers:
  - /github-read-file
  - "read this file from the repo"
  - "show me the current content of"
reads_from: any file path in the local repo
writes_to: sanctum/00 Inbox/github-read-file-[YYYY-MM-DD].md
model_preference: claude-sonnet
---

# github/read-file

## When to use

Use when you need to read the current on-disk content of a specific file
in the sanctum-bifrost workspace and produce a structured report of what
it contains. This is the infrastructure skill that lets agents verify what
is actually in the system before acting on it — as opposed to assuming
content from memory or prior context.

## What it does

1. Receives a file path relative to the sanctum-bifrost root
   (e.g. `bifrost/CODEX.md` or `sanctum/10 Knowledge/Design/runes/press.md`).
2. Reads the file from disk exactly as it exists.
3. Counts lines and captures the first non-empty line as a title reference.
4. Writes a structured report to `sanctum/00 Inbox/github-read-file-[YYYY-MM-DD].md`.
5. Reports: "Read [filename] · [N] lines · report written to Inbox."

## What it does NOT do

- Does not modify the file it reads.
- Does not summarise or paraphrase — the report contains the raw content
  verbatim, not an interpretation.
- Does not read multiple files in one invocation — one file per dispatch.
- Does not write to any location other than `sanctum/00 Inbox/`.
- Does not make judgments about whether the file content is correct.

## Inputs

**Required:**
- `file_path` — path to the file, relative to sanctum-bifrost root.
  Example: `bifrost/contracts/design-pipeline-contract.md`

**Optional:**
- `line_range` — start and end lines to read if only a section is needed.
  Format: `start:end`. Example: `1:50`. Defaults to full file.

## Output

One markdown report at `sanctum/00 Inbox/github-read-file-[YYYY-MM-DD].md`.

Format:
```markdown
# github/read-file — [filename]

**Date:** YYYY-MM-DD
**Path:** [full relative path]
**Lines:** [N]
**First line:** [first non-empty line of the file]

---

## Content

[raw file content, verbatim]
```

## Failure modes

**File not found:** Stop. Report the exact path that was not found and the
exact path that was provided. Do not guess an alternate location.

**File is binary or non-text:** Stop. Report the file type and that this
skill reads text files only.

**Scope creep:** If the task description implies reading multiple files,
reading a directory, or searching for content — stop. Those are different
skills. Report the boundary and await a more specific instruction.
