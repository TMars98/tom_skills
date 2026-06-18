# tom_skills

A small, personal Claude Code marketplace — the superpowers I actually use, packaged as one installable plugin (`tom`).

Four focused capabilities, fully **provider-neutral**: everything works off plain `git`, your repo's `CLAUDE.md`/linter configs, and (optionally) GitHub via `gh`. No Jira/Bitbucket or MCP servers required.

## Why this exists

1. **Repeatability.** A workflow should produce the same shape of work no matter who runs it — me, a teammate, or Claude on its own. Encoding the steps as skills means they don't get skipped or improvised.
2. **Reuse.** The reviewers and agents are shared building blocks, and the commands are just orchestration over them. Adding a new workflow usually means composing existing pieces in a new order, not writing fresh analysis logic.
3. **Provider-neutral.** Nothing here is tied to a specific issue tracker, host, or MCP server. If you have `git` (and optionally `gh` and `python3`), it works — so the same kit drops into any repo.

## Install

```
/plugin marketplace add TMars98/tom_skills
/plugin install tom@tom-skills
```

Then the commands below are available as `/tom:<command>`.

## What's in it

### Code review — `/tom:local-review`

A composite that fans out to **four single-dimension reviewer sub-agents in parallel**, aggregates their findings into one report, and offers to apply fixes at a severity threshold you pick. Each reviewer reads your `CLAUDE.md` and surrounding code first, so findings respect house style.

```
/tom:local-review                 # all uncommitted changes
/tom:local-review --staged        # staged only
/tom:local-review --range a..b    # a commit range
/tom:local-review with tests      # add the 4th test-adequacy pass
```

The reviewers are also usable on their own as agents:

| Agent | Dimension | Severities |
|-|-|-|
| `correctness-reviewer` | Logic errors, broken invariants, missing error handling, regressions | `blocker`/`major`/`minor`/`nit` |
| `quality-reviewer` | Readability, naming, duplication, convention adherence | `major`/`minor`/`nit` |
| `security-reviewer` | Injection, authn/authz, secrets, OWASP top-10 | `critical`/`high`/`medium`/`low`/`nit` |
| `test-reviewer` | Untested behavior, tests that can't fail, missing edge/error coverage | `major`/`minor`/`nit` |

All return structured JSON: `{findings: [{severity, file, line, finding, fix}], summary}`. Reviewers surface **everything** (no pre-filtering); the consumer triages.

### Duplicate-work check — `/tom:precheck`

Before you build something, check whether it already exists.

```
/tom:precheck add CSV export to the admin dashboard
```

Searches the repo for an existing implementation, recent git history, local/remote branches, and open GitHub PRs (`gh`), then returns a verdict: **LIKELY DUPLICATE** / **POSSIBLY ADDRESSED** / **APPEARS NOVEL**. Fully read-only; degrades gracefully outside a git repo or without `gh`.

### Live visual scratchpad — `/tom:visualize`

```
/tom:visualize a sequence diagram of the login flow
/tom:visualize stop
```

Spins up a no-install local server (Python stdlib only, `127.0.0.1` only) and a browser page that live-renders whatever Claude writes to it. Buttons can post events back (`tomSend(...)`) which Claude reacts to live — a true two-way canvas. Degrades to a terminal explanation if `python3` is missing.

## Requirements

- **`git`** — for `/tom:local-review` and `/tom:precheck`
- **`gh`** (optional) — enables the GitHub-PR signal in `/tom:precheck`
- **`python3`** (optional) — required only for `/tom:visualize`

No MCP servers required.

## Layout

```
tom_skills/
├── .claude-plugin/marketplace.json
└── plugins/tom/
    ├── .claude-plugin/plugin.json
    ├── hooks/hooks.json              # arms the visualize inbox click-hook
    ├── agents/                       # the 4 reviewer sub-agents
    ├── commands/                     # local-review, precheck, visualize
    └── skills/visual-scratchpad/     # stdlib server + viewer for /tom:visualize
```
