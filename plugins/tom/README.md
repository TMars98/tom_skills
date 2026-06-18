# tom

Tom's Claude Code toolkit. One plugin, a few sharp skills.

> Full usage and examples live in the [marketplace root README](../../README.md). This file is a thin index so it doesn't drift.

## Commands

| Command | What it does |
|-|-|
| `/tom:local-review [--staged \| --range a..b \| <path>] [with tests]` | Dispatches the four reviewers in parallel against local changes, aggregates, and offers in-place fixes. |
| `/tom:precheck <idea>` | Checks whether work already exists — repo impl, git history, branches, open GitHub PRs. Read-only. |
| `/tom:visualize [concept \| stop \| clear]` | Live browser scratchpad Claude can draw to, with bidirectional click events. |

## Agents (composable building blocks)

Diff-based reviewers dispatched by `/tom:local-review`, reusable on their own:

- `correctness-reviewer` · `quality-reviewer` · `security-reviewer` · `test-reviewer`

All return `{findings: [{severity, file, line, finding, fix}], summary}`.

## Auto-invoked skills

- `visual-scratchpad` — fires when a rendered view beats terminal prose; arms the scratchpad.

## Requirements

- `git` (reviews, precheck) · `gh` (optional, GitHub PR signal in precheck) · `python3` (optional, visualize only). No MCP servers needed.
