---
description: Review local changes for correctness only (logic errors, broken invariants, missing error handling, regressions). Returns a markdown report with all findings. Read-only.
argument-hint: [--staged | --uncommitted | --range <a>..<b> | <path>]
---

You are running **`/tom:correctness-review`** — a single-dimension, read-only review.

The user passed: `$ARGUMENTS`

## Step 1 — Resolve scope

Parse `$ARGUMENTS`:
- No arguments → `git diff HEAD` (all uncommitted: staged + unstaged) [default]
- `--staged` → `git diff --cached`
- `--uncommitted` → `git diff`
- `--range <a>..<b>` → `git diff <a>..<b>`
- A path or paths → `git diff HEAD -- <paths>`

Honor any freeform context (e.g. "ignore the test files"). If `$ARGUMENTS` describes what the change is *supposed* to do, treat that as optional spec/intent to pass through.

If not in a git repo (`git rev-parse --show-toplevel` fails), say so and stop. Run the diff; if empty, say "No changes to review." and stop. Capture `git rev-parse --show-toplevel` as the repo root.

## Step 2 — Dispatch the correctness reviewer

Make a single `Agent` call with `subagent_type="correctness-reviewer"`. Prompt:

```
repo_root: <path>

Scope: <description, e.g. "all uncommitted changes" / "staged only" / "files X,Y">

Diff:
<diff>

You have full local tool access (Read, Grep, Glob, Bash). Discover CLAUDE.md (root + nested near changed files) and surrounding conventions yourself before reviewing. Return findings as JSON per your output contract.
```

Include any supplied spec/intent as a fenced markdown block. Wait for the response; extract the trailing JSON object.

## Step 3 — Render the report

Parse the agent's JSON. Render:

```
## Correctness Review — <scope description>

**Files:** <count>

### Blockers
- `file:line` — finding. **Fix:** …
(or "_None._")

### Major
- `file:line` — finding. **Fix:** …

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
