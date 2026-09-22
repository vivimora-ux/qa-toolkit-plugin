# claude-code-plugins

A Claude Code **plugin marketplace repo** — not an application. There's no
build, no runtime, no source code to compile. The deliverable is markdown:
one marketplace manifest and one plugin, `qa-toolkit`, made of Skill
definitions and shared reference docs.

`qa-toolkit` helps a QA/QE team member build fast understanding of a
project, review PRs from a testing-risk lens, turn that analysis into test
plans and a full test-case inventory, and — its newest and only
code-writing skill — scaffold real, runnable automated test files from that
inventory.

## Repo layout

```
claude-code-plugins/
├── .claude-plugin/
│   └── marketplace.json
├── docs/
│   └── 0N-*-plan.md          # design docs for each build phase (Jira-story style)
├── TESTING.md                 # manual QA script covering all 5 skills
└── qa-toolkit-plugin/
    ├── .claude-plugin/
    │   └── plugin.json
    ├── skills/
    │   ├── rs-aut/SKILL.md
    │   ├── pr-explainer/SKILL.md
    │   ├── test-plan/SKILL.md
    │   ├── test-suite/SKILL.md
    │   └── test-automate/SKILL.md
    └── reference/
        ├── modifiers.md       # junior/mid/senior + visual/trace/risk — source of truth
        └── frameworks.md       # per-framework conventions for test-automate
```

Each skill also writes its output to a dated file under the *consuming*
project's own `docs/<skill>/` folder at runtime — that's a different `docs/`
than the one above, which holds this repo's own design docs.

## The modifier system

Two modifier categories are shared across skills as session-persistent
arguments: **skill level** (`junior`/`mid`/`senior`) and **entry point**
(`visual`/`trace`/`risk`). `test-automate` is the one asymmetric exception —
it reuses skill level only, to control comment density in generated code,
and has no use for entry point at all.

**`qa-toolkit-plugin/reference/modifiers.md` is the source of truth for
exact per-level/per-entry-point behavior.** Don't restate or paraphrase its
rules elsewhere (including here) — read it directly, since a second
paraphrased copy will drift out of sync with the real spec.

## Skill authoring conventions

- **Frontmatter**: `name`, a `description` that documents every modifier/
  argument the skill accepts, `disable-model-invocation: true`, and an
  `argument-hint` showing the accepted argument order.
- **Non-invention rule**: every skill after the first in the pipeline
  (`rs-aut` → `test-suite`/`test-plan` → `test-automate`, with
  `pr-explainer` a standalone `rs-aut`-fed PR reviewer that doesn't feed
  `test-plan`) treats its predecessor's doc as already-decided fact. If
  that doc doesn't exist yet, say so plainly and suggest running the
  missing skill — never analyze or invent the missing layer yourself.
- **Dated output files**: every skill run writes a markdown file under the
  target project's `docs/<skill>/<skill>_<identifier>_<date>.md`, leading
  with a header block (creation date, commands used) and updating that same
  file in place on same-day reruns rather than duplicating it. See
  `README.md` → "Saving a session to a file" for the exact per-skill naming.

## `pr-explainer`: the one skill with optional GitHub MCP enrichment

`pr-explainer` can optionally enrich a PR review with data the `gh` CLI
doesn't provide — review comments, CI/check and approval status, and
linked issues — via GitHub MCP. `gh` stays the primary source either
way; MCP only supplements it, folded into the existing seven-topic
structure (linked issues into change summary, review comments into risk
assessment, CI/approval status into non-functional considerations),
never a separate section of its own.

Off by default, and never proactively asked about — it turns on only
via an explicit `github` argument to `/pr-explainer`, after which
enablement persists across future runs on any PR/branch in that project
the same way `test-suite`'s Jira integration persists its project key.
Unlike Jira, there's no key or project to resolve: the PR is already
identified through this skill's own scope resolution, so enabling
GitHub MCP only means authenticating once and reading — never a
separate opt-in write step, since this skill has no write-back
concept.

Full behavior: `qa-toolkit-plugin/skills/pr-explainer/SKILL.md`.

## `test-automate`: the one skill that writes code

`test-automate` is the plugin's only skill that produces real files outside
`docs/` — generated test specs land in the target project's actual test
directory. This makes it the most relevant skill to the code-generation
agent work in this repo, and it carries a correspondingly stricter process:

- **Group before generating.** Every automatable case is assigned to a
  target spec file *before* any generation starts, so two agents never
  write the same file.
- **Agent fan-out.** The skill calls the plugin-bundled `test-spec-writer`
  agent (`qa-toolkit-plugin/agents/test-spec-writer.md`) once per target
  file, in parallel across files, via `subagent_type: test-spec-writer`.
  Each call is given its assigned cases, the exact target path (new vs.
  appending to an existing file), the resolved framework name, and the
  current comment-density level — `test-spec-writer` itself owns looking up
  `reference/frameworks.md` and adding the required traceability comment
  linking each generated test back to its source case.
- **Framework resolution order**: explicit argument > a prior run's
  persisted `Framework:` header (`docs/test-automate/test-automate_*.md`) >
  detected from `rs-aut`'s "Technology stack" section > ask once and
  persist the answer. Never pick a framework unilaterally when nothing is
  detected or persisted.
- **`reference/frameworks.md` is a starting point, not settled convention.**
  It's explicitly unreviewed until a senior compares real generated output
  against how a pilot project's engineers actually write tests. Treat it as
  provisional, and update it once that review happens rather than treating
  its current contents as final.

Full behavior: `qa-toolkit-plugin/skills/test-automate/SKILL.md`.

## `test-suite`: the one skill with an optional Jira integration

`test-suite` can optionally read from and write to Jira via the
already-installed Atlassian Rovo MCP server. This is unrelated to the
shared skill-level/entry-point modifier system above — it's about
*where source material comes from* (Jira requirements and acceptance
criteria, always alongside required `rs-aut` docs, never replacing
them) and *where output can optionally be pushed* (a single summary
tracking issue, only when explicitly asked), not about how
already-gathered content gets framed.

Jira integration is off by default and never proactively asked about —
it turns on only via an explicit `jira` argument to `/test-suite`,
after which the resolved project/epic key persists across future runs
the same way `test-automate`'s `Framework:` header does. Writing to
Jira is separately opt-in every time: it requires an explicit ask in
the session and never happens automatically, unlike this skill's
standing local markdown write.

Full behavior: `qa-toolkit-plugin/skills/test-suite/SKILL.md`.

## Verifying changes

There's no build/lint/test tooling in this repo — it's pure markdown, so
"testing" a skill change means walking through `TESTING.md`, which has a
numbered manual QA script covering defaults, modifier combinations, file
output/update-in-place behavior, and `test-automate`'s framework resolution
and comment-density differences for each skill.

## Status

Pilot / demo. Content should be reviewed by senior QA team members before
wider rollout, and the marketplace currently lives under a personal GitHub
account rather than a company org — treat conventions here (especially
`reference/frameworks.md`) as provisional, not settled, until that review
happens.
