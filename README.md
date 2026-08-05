# qa-toolkit

A Claude Code plugin that helps a QA/QE team member build fast, robust
understanding of a project's basics — project summary, architecture
overview, data flow and Data Cloud integrations, flow automation
requirements, testing strategy, and rollout plan — via the `rs-aut`
skill. It also includes a `pr-explainer` skill for reviewing a pull
request from a testing-risk lens: what changed, what's untested, and
what's fragile — a `test-plan` skill that turns that analysis into
a prioritized manual + automated test plan for one PR or feature area —
and a `test-suite` skill that reads across all of a project's `rs-aut`
docs to build a full, project-wide test case inventory, for projects
that don't have one yet.

## Install

From inside Claude Code, in the project you want to use it in:

```
/plugin marketplace add vivimora-ux/qa-toolkit-plugin
/plugin install qa-toolkit@qa-toolkit-plugin
```

## Update

When the plugin content changes, pull the latest version:

```
/plugin marketplace update
```

## Usage: project onboarding (`/rs-aut`)

Ask a question about the project, e.g.:

```
/rs-aut help me understand the checkout flow
```

Optionally add modifier arguments to set how the answer is framed and
how deep it goes. These stay in effect for the rest of the session,
until changed again — and are shared with the `pr-explainer` skill below,
so setting one carries over to the other:

**Entry point** — how the explanation is framed:
- `visual` — lead with structure/architecture
- `trace` — lead with a step-by-step walk-through
- `risk` — lead with what's fragile or historically problematic

**Skill level** — how much scaffolding is used:
- `junior` — define terms, explain why things matter, assume no context
- `mid` — default; define only uncommon terms
- `senior` — precise vocabulary, no definitions, straight to edge cases

Example:

```
/rs-aut senior risk the payments service
```

Skill level is self-set and can change any time — it's meant to be
updated as someone's familiarity with the project grows, not a fixed
label.

## Usage: PR review (`/pr-explainer`)

Review a pull request from a testing-risk lens — what changed, what's
untested, what's fragile — rather than a general code-quality review:

```
/pr-explainer 142
/pr-explainer https://github.com/org/repo/pull/142
/pr-explainer
```

- With a PR number or URL, it uses the GitHub CLI (`gh`) to pull the
  PR's description, commits, and diff.
- With no argument, it reviews the current branch's diff against the
  repo's default branch.

The same `junior` / `mid` / `senior` and `visual` / `trace` / `risk`
modifiers from above apply here too — `visual` leads with the
files/components touched, `trace` leads with a step-by-step code
walk-through, and `risk` leads with the risk assessment.

## Usage: test plan (`/test-plan`)

Turn a `pr-explainer` or `rs-aut` doc that's already been produced into a
concrete, prioritized manual + automated test plan — not a fresh
re-analysis of the PR or project:

```
/test-plan 142
/test-plan the checkout flow
/test-plan senior risk PR#42
/test-plan
```

- With a PR number/link or no argument on a branch with a diff, it's
  PR-scoped and reads the matching `docs/pr-explainer/` doc.
- With a feature/area name, or no argument and no diff, it's
  project-scoped and reads the matching `docs/onboarding/` doc.
- If neither doc exists yet, it says so and suggests running
  `/pr-explainer` or `/rs-aut` first rather than analyzing anything from
  scratch itself.

Automated test case suggestions default to whatever automation tooling
`rs-aut`'s doc found already in place for that layer, only deviating when
that doc gives an explicit reason to (no tooling yet, or the existing
tooling flagged as technical debt).

## Usage: full test-suite inventory (`/test-suite`)

`test-plan` is deliberately narrow: it scopes to one PR or one feature
area. `test-suite` is the project-wide counterpart — it reads across
*all* of a project's `rs-aut` docs (not just one) to build a full,
exhaustive test case inventory: every manual and automatable test case a
QA should have written for the whole application. Reach for `test-suite`
when a project has little or no existing test automation and there's no
single PR or area to scope a plan against yet; reach for `test-plan` once
there's a specific change or area in front of you.

```
/test-suite
/test-suite senior risk
```

- Looks for every `docs/onboarding/rs-aut_*.md` file in the project, and
  combines them if the project has been onboarded piecemeal across
  multiple areas or days.
- If no `rs-aut` docs exist yet, it says so and suggests running
  `/rs-aut` first, rather than analyzing the project from scratch itself.
- Names explicitly which feature areas or topics have no `rs-aut` coverage
  yet, so the inventory is honest about what's partial.
- Every case is tagged with a priority (highest-risk first, traced back to
  what `rs-aut` actually flagged) and a type (functional / edge / negative /
  integration).

The same `junior` / `mid` / `senior` and `visual` / `trace` / `risk`
modifiers are shared across all four skills.

## Saving a session to a file

Every answer is automatically saved (no need to ask) as a dated
markdown file:

- `/rs-aut` answers save under `docs/onboarding/` in the project
  being onboarded, e.g. `docs/onboarding/rs-aut_2026-07-28.md`.
- `/pr-explainer` answers save under `docs/pr-explainer/`, e.g.
  `docs/pr-explainer/pr-explainer_pr142_2026-07-28.md` (or
  `pr-explainer_<branch>_2026-07-28.md` for a local-diff review).
- `/test-plan` answers save under `docs/test-plan/`, e.g.
  `docs/test-plan/test-plan_pr142_2026-07-28.md` for a PR-scoped run, or
  `docs/test-plan/test-plan_2026-07-28.md` for a project/feature-scoped
  run.
- `/test-suite` answers save under `docs/test-suite/`, e.g.
  `docs/test-suite/test-suite_2026-07-28.md`. One file per day, project-wide.

Repeated answers on the same day, for the same topic/PR/branch, update
that same file rather than creating a new one. Each file leads with the
creation date and the full list of commands used to generate it, then
the content itself under clear headers — easy to skim in an editor like
VS Code instead of scrolling back through the terminal.

## Repo layout

```
aut-onboarding-marketplace/
├── .claude-plugin/
│   └── marketplace.json
└── qa-toolkit-plugin/
    ├── .claude-plugin/
    │   └── plugin.json
    ├── skills/
    │   ├── rs-aut/
    │   │   └── SKILL.md
    │   ├── pr-explainer/
    │   │   └── SKILL.md
    │   ├── test-plan/
    │   │   └── SKILL.md
    │   └── test-suite/
    │       └── SKILL.md
    ├── reference/
    │   └── modifiers.md   # shared junior/mid/senior + visual/trace/risk behavior
    └── README.md   (this file)
```

## Status

Pilot / demo. Content should be reviewed by senior team members before
wider rollout, and this marketplace currently lives under a personal
GitHub account (`vivimora-ux`) rather than a company org — fine for the
pilot, worth revisiting before full team distribution.
