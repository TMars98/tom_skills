# tom

Tom's Claude Code toolkit. One plugin, a few sharp skills.

> Full usage and examples live in the [marketplace root README](../../README.md). This file is a thin index so it doesn't drift.

## Commands

| Command | What it does |
|-|-|
| `/tom:local-review [--staged \| --range a..b \| <path>] [with tests]` | Dispatches the four reviewers in parallel against local changes, aggregates, and offers in-place fixes. |
| `/tom:correctness-review [<scope>]` | Single-dimension: logic errors, broken invariants, regressions. Read-only. |
| `/tom:quality-review [<scope>]` | Single-dimension: readability, naming, duplication, conventions. Read-only. |
| `/tom:security-review [<scope>]` | Single-dimension: injection, authn/authz, secrets, OWASP. Read-only. |
| `/tom:test-review [<scope>]` | Single-dimension: untested behavior, tests that can't fail, edge/error coverage. Read-only. |
| `/tom:precheck <idea>` | Checks whether work already exists — repo impl, git history, branches, open GitHub PRs. Read-only. |
| `/tom:visualize [concept \| stop \| clear]` | Live browser scratchpad Claude can draw to, with bidirectional click events. |

Scope flags for the review commands: `--staged` · `--uncommitted` · `--range a..b` · `<path>` (default: all uncommitted).

## Agents (composable building blocks)

Diff-based reviewers dispatched by `/tom:local-review`, reusable on their own:

- `correctness-reviewer` · `quality-reviewer` · `security-reviewer` · `test-reviewer`

All return `{findings: [{severity, file, line, finding, fix}], summary}`.

## Auto-invoked skills

- `visual-scratchpad` — fires when a rendered view beats terminal prose; arms the scratchpad.
- `frontend-router` — fires at the start of UI work; routes to the `frontend-design` plugin (notify-don't-force).

## Requirements

- `git` (reviews, precheck) · `gh` (optional, GitHub PR signal in precheck) · `python3` (optional, visualize only). No MCP servers needed.
