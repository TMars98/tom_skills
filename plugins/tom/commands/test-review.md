---
description: Review local changes for test adequacy only (untested behavior, tests that can't fail, missing edge/error coverage, flaky patterns). Returns a markdown report with all findings. Read-only.
argument-hint: [--staged | --uncommitted | --range <a>..<b> | <path>]
---

You are running **`/tom:test-review`** — a single-dimension, read-only review.

The user passed: `$ARGUMENTS`

## Step 1 — Resolve scope

Parse `$ARGUMENTS`:
- No arguments → `git diff HEAD` (all uncommitted: staged + unstaged) [default]
- `--staged` → `git diff --cached`
- `--uncommitted` → `git diff`
- `--range <a>..<b>` → `git diff <a>..<b>`
- A path or paths → `git diff HEAD -- <paths>`

Honor any freeform context.

If not in a git repo (`git rev-parse --show-toplevel` fails), say so and stop. Run the diff; if empty, say "No changes to review." and stop. Capture `git rev-parse --show-toplevel` as the repo root.

## Step 2 — Dispatch the test reviewer

Make a single `Agent` call with `subagent_type="test-reviewer"`. Prompt:

```
repo_root: <path>

Scope: <description, e.g. "all uncommitted changes" / "staged only" / "files X,Y">

Diff:
<diff>

You have full local tool access (Read, Grep, Glob, Bash). Discover CLAUDE.md testing rules, the test framework, and where tests live yourself. Grep for existing tests of changed symbols before reporting them untested. Return findings as JSON per your output contract.
```

Wait for the response; extract the trailing JSON object.

## Step 3 — Render the report

Parse the agent's JSON. Render:

```
## Test Coverage Review — <scope description>

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

Render `_None._` for empty sections rather than dropping them. (There is no blocker tier for test adequacy — a missing test isn't a production blocker.)

## Step 4 — Hand off

End the response. This command is **read-only** — do NOT write tests or anything else. The user decides what to do (or runs `/local-review with tests` to get the fix/test-writing flow).

If the agent returned malformed JSON, render its raw output under a "Reviewer output" section with a note that parsing failed.
