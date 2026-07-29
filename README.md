# qa-toolkit

A Claude Code plugin that helps a QA/QE team member build fast, robust
understanding of a project's basics: project summary, architecture
overview, data flow and Data Cloud integrations, flow automation
requirements, testing strategy, and rollout plan. It also includes a
`pr-review` skill for reviewing a pull request from a testing-risk
lens: what changed, what's untested, and what's fragile.

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

Optionally set how the answer is framed and how deep it goes, before or
alongside your question. These stay in effect for the rest of the
session, until changed again — and are shared with the `pr-review`
skill below, so setting one carries over to the other:

**Entry point** — how the explanation is framed:
- `/visual` — lead with structure/architecture
- `/trace` — lead with a step-by-step walk-through
- `/risk` — lead with what's fragile or historically problematic

**Skill level** — how much scaffolding is used:
- `/junior` — define terms, explain why things matter, assume no context
- `/mid` — default; define only uncommon terms
- `/senior` — precise vocabulary, no definitions, straight to edge cases

Example:

```
/senior /risk
/rs-aut the payments service
```

Skill level is self-set and can change any time — it's meant to be
updated as someone's familiarity with the project grows, not a fixed
label.

## Usage: PR review (`/pr-review`)

Review a pull request from a testing-risk lens — what changed, what's
untested, what's fragile — rather than a general code-quality review:

```
/pr-review 142
/pr-review https://github.com/org/repo/pull/142
/pr-review
```

- With a PR number or URL, it uses the GitHub CLI (`gh`) to pull the
  PR's description, commits, and diff.
- With no argument, it reviews the current branch's diff against the
  repo's default branch.

The same `/junior` / `/mid` / `/senior` and `/visual` / `/trace` /
`/risk` commands from above apply here too — `/visual` leads with the
files/components touched, `/trace` leads with a step-by-step code
walk-through, and `/risk` leads with the risk assessment.

## Saving a session to a file

Every answer is automatically saved (no need to ask) as a dated
markdown file:

- `/rs-aut` answers save under `docs/onboarding/` in the project
  being onboarded, e.g. `docs/onboarding/rs-aut_2026-07-28.md`.
- `/pr-review` answers save under `docs/pr-reviews/`, e.g.
  `docs/pr-reviews/pr-review_pr142_2026-07-28.md` (or
  `pr-review_<branch>_2026-07-28.md` for a local-diff review).

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
    │   └── pr-review/
    │       └── SKILL.md
    ├── commands/
    │   ├── visual.md
    │   ├── trace.md
    │   ├── risk.md
    │   ├── junior.md
    │   ├── mid.md
    │   └── senior.md
    └── README.md   (this file)
```

## Status

Pilot / demo. Content should be reviewed by senior team members before
wider rollout, and this marketplace currently lives under a personal
GitHub account (`vivimora-ux`) rather than a company org — fine for the
pilot, worth revisiting before full team distribution.
