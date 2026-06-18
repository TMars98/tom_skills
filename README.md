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

Each dimension is also its own **single-dimension, read-only command** — run just the one you want (same scope flags as `local-review`):

```
/tom:correctness-review        # logic errors, broken invariants, regressions
/tom:quality-review            # readability, naming, duplication, conventions
/tom:security-review           # injection, authn/authz, secrets, OWASP
/tom:test-review               # untested behavior, tests that can't fail
```

These just print a report — they never apply fixes (use `/tom:local-review` for the fix-application flow). Each is backed by a reusable agent you can also dispatch directly (e.g. "use the quality-reviewer agent on my staged changes"):

| Command | Agent | Dimension | Severities |
|-|-|-|-|
| `/tom:correctness-review` | `correctness-reviewer` | Logic errors, broken invariants, missing error handling, regressions | `blocker`/`major`/`minor`/`nit` |
| `/tom:quality-review` | `quality-reviewer` | Readability, naming, duplication, convention adherence | `major`/`minor`/`nit` |
| `/tom:security-review` | `security-reviewer` | Injection, authn/authz, secrets, OWASP top-10 | `critical`/`high`/`medium`/`low`/`nit` |
| `/tom:test-review` | `test-reviewer` | Untested behavior, tests that can't fail, missing edge/error coverage | `major`/`minor`/`nit` |

All reviewers return structured JSON: `{findings: [{severity, file, line, finding, fix}], summary}`, surface **everything** (no pre-filtering), and let the consumer triage.

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

### Frontend routing — `frontend-router` (auto-invoked)

Not a command — a skill that fires automatically at the start of user-facing UI work (new components, pages, redesigns, "make it look like…"). It routes the work to the [`frontend-design`](https://docs.claude.com) plugin, which generates distinctive, production-grade interfaces.

It's *notify-don't-force*: if `frontend-design` is installed it hands off to it; if not, it tells you how to install it and lets you proceed either way. Install the engine with:

```
/plugin install frontend-design@claude-plugins-official
```

The router never blocks and never force-installs; it skips itself for bug fixes, refactors, and non-visual work.

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
    ├── commands/                     # local-review, *-review, precheck, visualize
    └── skills/
        ├── visual-scratchpad/        # stdlib server + viewer for /tom:visualize
        └── frontend-router/          # auto-routes UI work to frontend-design
```
