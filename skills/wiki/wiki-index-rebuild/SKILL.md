---
name: wiki/index-rebuild
version: 1.0.0
type: producer
domain: wiki
tier: action
triggers:
  - /wiki-index-rebuild
  - "update the index"
  - "rebuild index after changes"
reads_from: list of changed files (provided by caller) + sanctum/01 Indexes/index.md
writes_to: sanctum/01 Indexes/index.md + sanctum/01 Indexes/log.md
model_preference: claude-sonnet
---

# wiki/index-rebuild

## When to use

Use immediately after any implementation session that added, moved, or
removed files in Sanctum or Bifrost. The caller provides the explicit list
of files that changed in the current pass — this skill does not scan the
whole tree. It updates only the index entries for those specific files and
appends one log entry recording the session.

This is the session-close skill. Dispatch it as the last step of every
Codex implementation session.

## What it does

1. Receives the list of changed files from the caller (paths relative to
   sanctum-bifrost root).
2. Reads `sanctum/01 Indexes/index.md` to find existing entries.
3. For each file in the changed list:
   - If the file was **added**: adds a new row to the appropriate section
     in index.md with the file path and `[pending description]` as
     placeholder. Does not read file content to generate descriptions.
   - If the file was **modified**: updates the `Updated` date in its
     existing row. Does not change the description.
   - If the file was **deleted**: marks the row with `[removed YYYY-MM-DD]`
     in the description column. Does not delete the row.
4. Appends one entry to `sanctum/01 Indexes/log.md` in this format:
   `## [YYYY-MM-DD] wiki/index-rebuild | [N added, N modified, N removed] · session: [session-label]`
5. Reports: "Index updated · [N] rows changed · log entry appended."

## What it does NOT do

- Does not scan the full Sanctum tree — only processes the provided file list.
- Does not read file contents to generate descriptions — descriptions are
  `[pending description]` until a human fills them in.
- Does not modify any file other than `index.md` and `log.md`.
- Does not delete rows from index.md — it marks them removed instead,
  preserving the audit trail.
- Does not run on a schedule — always dispatched manually by the human
  after an implementation session.

## Inputs

**Required:**
- `changed_files` — list of file paths that changed in the current session.
  Format: one path per line, relative to sanctum-bifrost root. Annotated
  with the change type:
  ```
  added:    bifrost/CODEX.md
  added:    sanctum/99 Templates/SKILL.md
  modified: sanctum/01 Indexes/index.md
  removed:  sanctum/99 Templates/.gitkeep
  ```

**Required:**
- `session_label` — a short label for the log entry identifying what this
  session was. Example: `phase-0-templates` or `github-skills-added`.

## Output

Two files modified in place:
- `sanctum/01 Indexes/index.md` — rows added, updated, or marked removed.
- `sanctum/01 Indexes/log.md` — one new entry appended at the bottom.

## Index format

Index.md is organised by top-level folder. Each section has a table:

```markdown
## [Folder Name]

| File | Description | Updated |
|---|---|---|
| `relative/path/file.md` | [pending description] | YYYY-MM-DD |
```

New rows are appended to the bottom of the relevant section's table.
If a section does not yet exist for a folder, create it with the header
and an empty table.

## Failure modes

**No changed_files provided:** Stop. Report that this skill requires an
explicit file list. Do not scan the tree.

**No session_label provided:** Stop. Report that a session label is
required for the log entry.

**index.md does not exist:** Create it from scratch with the standard
format. Add a header line:
`# Sanctum Index — last rebuilt [YYYY-MM-DD]`
Then proceed with the provided file list.

**File in changed_files list is in Bifrost, not Sanctum:** Still index it.
Bifrost files belong in a `## Bifrost` section in index.md. Create the
section if it does not exist.

**Scope creep:** If the task implies reading file contents, generating
summaries, or updating any file other than index.md and log.md — stop.
Those are wiki/ingest responsibilities. Report the boundary.
