# aut-onboarding

A Claude Code plugin that helps a QA/QE team member build fast, robust
understanding of a project's basics: project summary, architecture
overview, data flow and Data Cloud integrations, flow automation
requirements, testing strategy, and rollout plan.

## Install

From inside Claude Code, in the project you want to use it in:

```
/plugin marketplace add vivimora-ux/aut-onboarding-marketplace
/plugin install aut-onboarding@aut-onboarding-marketplace
```

## Update

When the plugin content changes, pull the latest version:

```
/plugin marketplace update
```

## Usage

Ask a question about the project, e.g.:

```
/aut-onboarding help me understand the checkout flow
```

Optionally set how the answer is framed and how deep it goes, before or
alongside your question. These stay in effect for the rest of the
session, until changed again:

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
/aut-onboarding the payments service
```

Skill level is self-set and can change any time — it's meant to be
updated as someone's familiarity with the project grows, not a fixed
label.

## Saving a session to a file

Every answer is automatically saved (no need to ask) as a dated markdown
file under `docs/onboarding/` in the project being onboarded, e.g.
`docs/onboarding/aut-onboarding_2026-07-28.md`. Repeated answers on the
same day update that same file rather than creating a new one. The file
leads with the creation date and the full list of commands used to
generate it, then the onboarding content itself under clear headers —
easy to skim in an editor like VS Code instead of scrolling back through
the terminal.

## Repo layout

```
aut-onboarding-marketplace/
├── .claude-plugin/
│   └── marketplace.json
└── aut-onboarding-plugin/
    ├── .claude-plugin/
    │   └── plugin.json
    ├── skills/
    │   └── aut-onboarding/
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
