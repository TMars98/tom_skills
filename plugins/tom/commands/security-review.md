---
description: Review local changes for security issues only (injection, authn/authz, secrets, OWASP top-10). Returns a markdown report with all findings including low-severity observations. Read-only.
argument-hint: [--staged | --uncommitted | --range <a>..<b> | <path>]
---

You are running **`/tom:security-review`** — a single-dimension, read-only review.

The user passed: `$ARGUMENTS`

## Step 1 — Resolve scope

Parse `$ARGUMENTS`:
- No arguments → `git diff HEAD` (all uncommitted: staged + unstaged) [default]
- `--staged` → `git diff --cached`
- `--uncommitted` → `git diff`
- `--range <a>..<b>` → `git diff <a>..<b>`
- A path or paths → `git diff HEAD -- <paths>`

Honor any freeform context.

If not in a git repo (`git rev-parse --show-toplevel` fails), say so and stop. Run the diff; if empty, say "No changes to review." and stop. Capture `git rev-parse --show-toplevel` as the repo root. Also run `git status --short` and flag any modified/staged `.env`, `appsettings*`, `secrets*`, or credential files.

## Step 2 — Dispatch the security reviewer

Make a single `Agent` call with `subagent_type="security-reviewer"`. Prompt:

```
repo_root: <path>

Scope: <description, e.g. "all uncommitted changes" / "staged only" / "files X,Y">

Diff:
<diff>

git status:
<git status --short output>

You have full local tool access (Read, Grep, Glob, Bash). Discover CLAUDE.md, the framework/language, and surrounding code yourself before reviewing. Return findings as JSON per your output contract.
```

Wait for the response; extract the trailing JSON object.

## Step 3 — Render the report

Parse the agent's JSON. Render:

```
## Security Review — <scope description>

**Files:** <count>

### Critical
- `file:line` — finding (with exploit path). **Fix:** …
(or "_None._")

### High
- `file:line` — finding. **Fix:** …

### Medium
- `file:line` — finding. **Fix:** …

### Low
- `file:line` — finding. **Fix:** …

### Nits
- `file:line` — finding. **Fix:** …

**Summary:** <one-line take from agent>
```

Render `_None._` for empty sections rather than dropping them.

## Step 4 — Hand off

End the response. This command is **read-only** — do NOT apply fixes or write anything. The user decides what to do (or runs `/local-review` to get the fix-application flow).

If the agent returned malformed JSON, render its raw output under a "Reviewer output" section with a note that parsing failed.
