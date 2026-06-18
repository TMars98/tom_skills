---
description: Check whether a piece of work already exists before starting it — searches the repo for an existing implementation, recently-merged git history, local/remote branches, and open GitHub PRs, then renders a verdict. Read-only.
argument-hint: "<free-text idea or feature> [skip repo | skip git | skip prs]"
---

You are running **`/tom:precheck`**. Decide whether a piece of work is worth doing by checking if it already exists — as code in the repo, as a recently-merged commit, as an in-flight branch, or as an open PR. **Read-only**: this command never writes, commits, or modifies anything.

The user passed: `$ARGUMENTS`

## Step 1 — Parse arguments

From `$ARGUMENTS`:
- **Query text** — the description of the work to check (a feature, fix, or idea the user is about to build). This drives keyword extraction.
- **Scope modifiers** (case-insensitive; strip from the query text before using it):
  - `skip repo` → suppress the repo-implementation signal (Step 3a)
  - `skip git` → suppress the git-history signal (Step 3c)
  - `skip prs` → suppress the PR signal (Step 3b)

If `$ARGUMENTS` is empty, exit with: "I need a description of the work. Try `/tom:precheck add CSV export to the admin dashboard`."

Extract distinctive **keywords** from the query: domain nouns, feature names, likely function/route/symbol names. Drop stopwords. Prefer specific terms over generic ones — keyword quality drives recall.

Determine the repo root with `git rev-parse --show-toplevel`. If that fails, you're not in a git repo: skip Steps 3b and 3c, and note it in the report.

## Step 2 — Gather signals

Each signal is gathered independently. Any failure degrades to a `_<signal> failed: <reason>_` line in that section — it never aborts the command. Skip any signal the user suppressed.

### 3a — Repo: existing implementation

Skip if `skip repo` was set. Use `Grep`/`Glob` to locate candidate files for the keywords (routes, function/symbol names, domain nouns), then `Read` the top hits to judge whether the functionality already exists. Record `file:line symbol` for each genuine hit. A keyword appearing in an unrelated context is not evidence — use judgment.

### 3b — In-flight: open PRs and branches

Skip if `skip prs` was set or not in a git repo.

- **Branches:** run `git branch -a` and keep branches whose names match the keywords. Record local vs remote.
- **Open PRs:** if `gh` is available (`command -v gh`) and the origin is GitHub, run `gh pr list --state open --search "<keywords>" --json number,title,headRefName,author --limit 20` and keep matches. If `gh` is absent or the repo isn't on GitHub, render `_PR search unavailable (no gh / not GitHub)._` and continue.

### 3c — Merged: recent git history

Skip if `skip git` was set or not in a git repo. Run `git log --oneline --grep="<term>" -i` for one or a few representative terms (cap output to ~15 lines) to find already-merged work. Record `<short-sha> <subject> (<relative date>)`.

## Step 3 — Synthesize and render

Set the overall verdict. Verdicts are ordered — evaluate top-down and emit the first that matches:

- **LIKELY DUPLICATE** — the feature appears already shipped (a real repo implementation in 3a, or a clearly-matching merged commit in 3c).
- **POSSIBLY ADDRESSED** — partial repo hits, or overlapping in-flight work (matching branch or open PR) but no shipped implementation.
- **APPEARS NOVEL** — nothing material across all signals.

Render (plain text tags, no emoji):

```
## Precheck — "<query text>"

**Verdict:** LIKELY DUPLICATE
<one-line synthesis>

### Repo implementation
- src/export/csv.ts:42  exportToCsv()  — feature appears implemented
_No matching implementation found._

### In-flight work
- PR #210  Add CSV export  (feat/csv-export · @adev)
- branch  feature/csv-export  (remote)
_None._

### Recently merged
- abc1234  feat: add CSV export  (3w ago)
_None._
```

Render the appropriate `_None._` / `_No matching implementation found._` line under any section with no hits. When a signal was skipped or unavailable, render `_Skipped (<reason>)._` or `_<reason>._` instead.

## Step 4 — Operator notes

- **Read-only. Always.** Never create branches, commits, PRs, or files; never modify ticket/issue state. `precheck` only reports.
- Repo, git, and PR signals skip gracefully outside a git repo or when `gh` is unavailable.
- Keyword extraction quality drives recall — pick distinctive terms over generic ones.
- The PR signal is scoped to the current repo's origin; there's no org-wide scan.
