---
name: github/read-diff
version: 1.0.0
type: producer
domain: github
tier: action
triggers:
  - /github-read-diff
  - "what changed in this commit"
  - "show me the diff for"
  - "what changed since"
reads_from: local git repository at a specified ref range or commit
writes_to: sanctum/00 Inbox/github-read-diff-[YYYY-MM-DD].md
model_preference: claude-sonnet
---

# github/read-diff

## When to use

Use when you need a structured report of what changed between two points in
the git history of a repo in the sanctum-bifrost workspace — either between
two refs, between a commit and HEAD, or for a single commit. The primary use
case in this system is giving design/curator and other pipeline skills
visibility into what has changed in a project since a Design Contract was
last instanced. Without this visibility, pipeline skills operate without
knowing the current state of the work.

## What it does

1. Receives a repo path and a ref range (e.g. `HEAD~1..HEAD`, a commit SHA,
   or two branch names).
2. Runs the diff for that range in the specified repo.
3. For each changed file, records: filename, change type (added / modified /
   deleted / renamed), and the diff content.
4. Writes a structured report to
   `sanctum/00 Inbox/github-read-diff-[YYYY-MM-DD].md`.
5. Reports: "Diff read · [N] files changed · [N additions, N deletions] ·
   report written to Inbox."

## What it does NOT do

- Does not modify any file in the repo it reads.
- Does not interpret what the changes mean or recommend next steps.
- Does not run diffs across two different repos — one repo per dispatch.
- Does not include binary file contents — binary files are listed by name
  and change type only.
- Does not write to any location other than `sanctum/00 Inbox/`.

## Inputs

**Required:**
- `repo_path` — path to the repo, relative to sanctum-bifrost root.
  Example: `bifrost` or `sanctum` or `runestone`.
- `ref_range` — the git ref range to diff.
  Examples: `HEAD~1..HEAD` (last commit), `abc1234..def5678` (two SHAs),
  `HEAD` (last commit, shorthand for `HEAD~1..HEAD`),
  `main..feature-branch` (branch comparison).

**Optional:**
- `path_filter` — limit the diff to files matching a path prefix or glob.
  Example: `skills/design/` or `*.md`. Defaults to all files.
- `context_lines` — number of context lines around each change. Defaults
  to 3. Set to 0 for minimal output on large diffs.

## Output

One markdown report at `sanctum/00 Inbox/github-read-diff-[YYYY-MM-DD].md`.

Format:
```markdown
# github/read-diff — [repo] [ref_range]

**Date:** YYYY-MM-DD
**Repo:** [repo_path]
**Ref range:** [ref_range]
**Summary:** [N] files changed · [N] insertions · [N] deletions

---

## Changed files

| File | Change | +/- |
|---|---|---|
| `path/to/file.md` | modified | +12 / -3 |
| `path/to/new.md` | added | +45 / -0 |
| `path/to/old.md` | deleted | +0 / -20 |

---

## Diffs

### `path/to/file.md` (modified)

\```diff
[raw diff content]
\```

### `path/to/new.md` (added)

\```diff
[raw diff content]
\```
```

Binary files are listed in the Changed files table with change type but
have no diff section.

## Failure modes

**No git history (empty repo or no commits):** Stop. Report that the repo
has no commits and that a diff cannot be produced. Do not attempt to
diff the working tree against nothing.

**Invalid ref range:** Stop. Report the exact ref range that failed and
the error message from git. Do not guess an alternate range.

**Ref range produces empty diff:** Write the report with an empty
Changed files table and a note: "No changes found in this range."
This is a valid result, not an error.

**Diff output exceeds 2000 lines:** Write the summary table and the first
500 lines of diff content, then append a note: "Diff truncated at 500 lines.
Use path_filter to narrow the range." Do not silently omit content.

**Scope creep:** If the task implies reading file contents outside the diff,
comparing across two repos, or generating a changelog narrative — stop.
Those are different skills. Report the boundary.
