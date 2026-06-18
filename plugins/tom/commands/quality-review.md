---
description: Review local changes for code quality only (readability, naming, duplication, convention adherence). Returns a markdown report with all findings including nits. Read-only.
argument-hint: [--staged | --uncommitted | --range <a>..<b> | <path>]
---

You are running **`/tom:quality-review`** — a single-dimension, read-only review.

The user passed: `$ARGUMENTS`

## Step 1 — Resolve scope

Parse `$ARGUMENTS`:
- No arguments → `git diff HEAD` (all uncommitted: staged + unstaged) [default]
- `--staged` → `git diff --cached`
- `--uncommitted` → `git diff`
- `--range <a>..<b>` → `git diff <a>..<b>`
- A path or paths → `git diff HEAD -- <paths>`

Honor any freeform context (e.g. "ignore the test files").

If not in a git repo (`git rev-parse --show-toplevel` fails), say so and stop. Run the diff; if empty, say "No changes to review." and stop. Capture `git rev-parse --show-toplevel` as the repo root.

## Step 2 — Dispatch the quality reviewer

Make a single `Agent` call with `subagent_type="quality-reviewer"`. Prompt:

```
repo_root: <path>

Scope: <description, e.g. "all uncommitted changes" / "staged only" / "files X,Y">

Diff:
<diff>

You have full local tool access (Read, Grep, Glob, Bash). Discover CLAUDE.md and convention files (.editorconfig, .eslintrc*, pyproject.toml, etc.) and surrounding code patterns yourself before reviewing. Return findings as JSON per your output contract.
```

Wait for the response; extract the trailing JSON object.

## Step 3 — Render the report

Parse the agent's JSON. Render:

```
## Quality Review — <scope description>

**Files:** <count>

### Major
- `file:line` — finding. **Fix:** …
(or "_None._")

### Minor
- `file:line` — finding. **Fix:** …

### Nits
- `file:line` — finding. **Fix:** …

**Summary:** <one-line take from agent>
```

Render `_None._` for empty sections rather than dropping them.

## Step 4 — Hand off

End the response. This command is **read-only** — do NOT apply fixes or write anything. The user decides what to do (or runs `/local-review` to get the fix-application flow).

If the agent returned malformed JSON, render its raw output under a "Reviewer output" section with a note that parsing failed.
