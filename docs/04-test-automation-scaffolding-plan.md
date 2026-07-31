# [Story] Test-suite automation scaffolding skill (Phase 4)

**Type:** Story (part of the "Personalized schema for accelerating AUT
understanding & judgment" epic)
**Priority:** Medium
**Reporter:** _(unassigned — draft, no ticket number yet)_
**Team:** QA / QE
**Labels:** `ai-tooling`, `qa-enablement`, `claude-code`, `test-automation`

---

## Summary

Add a new Skill, `test-automate`, to the existing `qa-toolkit` plugin
that takes the test case inventory `test-suite` (Phase 3) already
produced and generates real, runnable automated test code for a chosen
framework — Playwright, WebdriverIO, or another supported option. This
is explicitly a follow-up phase: it depends on `test-suite` existing and
having already run, and it depends on a framework decision being made
(by the person, or defaulted to whatever `rs-aut` already found in
place) before any code is generated.

This is the first skill in the plugin that writes code rather than
markdown analysis, which changes both its risk profile and its file
output model.

## Background

`test-suite` (Phase 3) produces a full, prioritized test case inventory
in plain language — useful on its own for manual testing, but for a
project with little or no automation, someone still has to translate
each case into actual test code by hand. This phase closes that gap:
once the inventory exists and a framework is chosen, `test-automate`
scaffolds the actual spec files, so automation coverage can start from a
generated baseline instead of a blank project.

This phase is intentionally sequenced after Phase 3 lands and gets
piloted — generating code well depends on the inventory being
trustworthy first, and framework/tool choice is a separate decision the
team should make deliberately rather than as a side effect of a single
skill run.

## Decisions made so far

- **Framework choice is explicit and persists, like skill level.** The
  person sets it once (or it's defaulted from `rs-aut`'s detected tech
  stack) and it carries over for the project, the same way skill-level
  commands persist for a session today. A new command layer
  (`/playwright`, `/webdriverio`, etc.) is the current leading idea,
  though this needs review since these commands operate at a different
  scope (project-persistent) than the existing session-persistent
  `/junior`/`/visual`-style modifiers — see open questions.
- **This does not replace `test-suite`.** `test-automate` only ever reads
  a `test-suite` doc (or a `test-plan` doc, for a narrower PR/area-scoped
  automation pass) — it doesn't invent test cases itself.
- **Scope for v1: new/greenfield automation only.** This phase targets
  projects with little or no existing automated tests. Retrofitting or
  refactoring an already-substantial existing test suite is out of scope
  for now (see Non-goals).

## Goals

- New Skill: `test-automate`, added to the existing `qa-toolkit` plugin.
- Reads a `docs/test-suite/` doc (project-wide) or a `docs/test-plan/`
  doc (PR/area-scoped) — whichever the person points it at — and treats
  its test cases as already-decided fact, same rule `test-plan` follows
  for `pr-explainer`/`rs-aut` docs today.
- Generates real spec files in the chosen framework, covering the
  automatable cases from the source doc (manual-only cases are skipped
  and noted, not force-converted).
- Framework/tool selection:
  - Defaults to whatever `rs-aut`'s "Technology stack" section already
    found in place, consistent with how `test-plan` already picks
    automation tooling.
  - If nothing is detected, asks once which framework to use, then
    persists that choice for the project going forward.
- Output includes, per generated spec: a comment linking back to the
  originating test case in the source doc, so traceability between the
  human-readable case and the generated code is never lost.
- Reuses `/junior` / `/mid` / `/senior` for how much explanatory comment
  accompanies generated code (e.g. `/junior` output includes more
  inline comments explaining each assertion; `/senior` output is
  comment-light, idiomatic code only).

## Non-goals

- Not choosing a framework on the team's behalf — the person (or a
  detected existing stack) decides; this skill never silently picks one
  for a project with no existing tooling and no stated preference.
- Not retrofitting or refactoring an existing, substantial automated
  test suite — v1 targets greenfield/low-automation projects only, per
  the decision above.
- Not running the generated tests or reporting pass/fail — this skill
  generates code; running it is a normal dev/CI concern outside the
  plugin.
- Not maintaining generated tests over time (e.g. auto-updating specs as
  the app changes) — this is a one-time scaffold from a point-in-time
  inventory, not an ongoing sync.
- Not inventing test cases that aren't in the source `test-suite` or
  `test-plan` doc — same non-invention rule as every other skill in the
  plugin.

## Proposed solution

### File structure additions

```
qa-toolkit-plugin/
├── .claude-plugin/
│   └── plugin.json          # version bump, updated description
├── skills/
│   ├── rs-aut/
│   ├── pr-explainer/
│   ├── test-plan/
│   ├── test-suite/
│   └── test-automate/
│       └── SKILL.md         # NEW
├── commands/                # possible new framework-choice commands,
│                             #  pending the open question below
├── reference/
│   ├── modifiers.md         # unchanged
│   └── frameworks.md        # NEW — per-framework conventions (file
│                             #  layout, naming, assertion style) so
│                             #  generated code matches what a project
│                             #  using that framework would expect
└── README.md                # updated with /test-automate usage
```

### The Skill (`test-automate`)

When invoked:

1. **Resolve the source doc.** If pointed at a PR/area, read the
   matching `docs/test-plan/` doc. Otherwise, read the most recent
   `docs/test-suite/` doc. If neither exists, say so plainly and suggest
   running `/test-suite` or `/test-plan` first — same non-invention
   pattern as `test-plan` follows today.
2. **Resolve the framework.** Use the detected/persisted choice, or ask
   once if nothing is set, per Goals above.
3. **Generate specs.** For each automatable case in the source doc,
   produce a spec file (or add a test case to an existing spec file, if
   one already exists for that area/component) following that
   framework's idiomatic conventions, per `reference/frameworks.md`.
4. **Report what was skipped.** Any case marked manual-only in the
   source doc is listed as skipped, with the reason, rather than
   silently omitted.

### Session/project modifiers

- `/junior` / `/mid` / `/senior` — control comment density and
  explanatory depth in generated code, same mechanism as elsewhere.
- Framework choice — persists at the project level (not just the
  session), since switching frameworks mid-project is a much bigger
  decision than switching explanation depth. Exact command mechanism
  TBD — see open questions.

### File output

- **Location** — generated specs go into the project's existing test
  directory convention for the chosen framework (e.g. `tests/` or
  `e2e/` for Playwright, `test/specs/` for WebdriverIO) — not a
  `docs/` folder, since this is real code, not an analysis doc.
- A short markdown summary (`docs/test-automate/test-automate_<date>.md`)
  is still written, listing what was generated, what was skipped and
  why, and the source doc(s) used — consistent with the rest of the
  plugin's habit of leaving a readable trail, but the actual deliverable
  here is the code itself.

## Acceptance criteria

- [ ] `test-automate/SKILL.md` drafted, correctly reading either a
      `test-suite` or `test-plan` doc as source
- [ ] Framework detection correctly defaults to what `rs-aut` already
      found in place, for at least one real project
- [ ] No-detected-framework case asks once and persists the answer for
      later runs on the same project
- [ ] Generated specs are runnable (compile/parse correctly) in at least
      one supported framework, verified against a real low-automation
      pilot project
- [ ] Manual-only cases are correctly skipped and listed, not
      force-converted or silently dropped
- [ ] `/junior` / `/mid` / `/senior` produce meaningfully different
      comment density in generated code
- [ ] Each generated spec traces back to its source test case via an
      in-code comment/reference
- [ ] Reviewed by a senior for whether generated code matches how the
      team actually structures tests for that framework
- [ ] Plugin version bumped and pushed; a teammate confirms
      `/plugin marketplace update` picks up the new skill

## Open questions / risks

- **Framework-choice command mechanism is undecided.** Unlike
  `/junior`/`/visual`-style modifiers, which reset per session by
  design, framework choice needs to persist at the project level. Worth
  deciding during implementation whether this is new commands
  (`/playwright`, `/webdriverio`), an argument to `/test-automate`
  itself (`/test-automate playwright`), or something read from a config
  file in the target project — each has different discoverability and
  persistence trade-offs.
  Update: since Claude Code now unifies commands and skills, and
  `.mcp.json`/plugin config already sets project-level state today, a
  small config file written by this skill's first run (e.g. under
  `docs/test-automate/` or a plugin-managed dotfile) may be simpler than
  new top-level commands — needs a decision, not just a default.
- **Code quality and idiom risk.** Generated code that doesn't match a
  team's actual conventions (page-object patterns, fixture setup, naming)
  creates cleanup work instead of saving it. `reference/frameworks.md`
  needs real review against how the pilot project's engineers actually
  write tests, not generic framework-tutorial style.
- **Where this fits with existing CI/test infra.** If the pilot project
  already has a CI pipeline expecting a certain test file location or
  naming convention, generated specs need to land there correctly or
  they'll be invisible to CI — worth confirming during pilot, not
  assumed.
- **Scope creep risk.** Because this skill writes code (not just
  analysis), it's tempting to extend it toward fixing/refactoring
  existing tests later. Explicitly out of scope for v1 (see Non-goals) —
  worth reinforcing at review time if pilot feedback pushes toward this.

## Suggested subtasks

1. Draft `test-automate/SKILL.md`
2. Draft `reference/frameworks.md`, starting with Playwright and
   WebdriverIO conventions
3. Decide and implement the framework-choice persistence mechanism
   (see open question above)
4. Generate specs against one real low-automation pilot project; verify
   they run/compile
5. Internal review with 2+ seniors — compare generated code style
   against how the team actually writes tests
6. Revise based on review feedback
7. Bump plugin version, update README, push to marketplace repo
8. Confirm update path works for a teammate via `/plugin marketplace
   update`

## Definition of done

`test-automate` is drafted, correctly reads a `test-suite` or
`test-plan` doc as its source, generates runnable test code in at least
one supported framework (defaulting to whatever `rs-aut` already found
in place, or asking once and persisting the answer), has been reviewed
by senior team members against a real low-automation pilot project for
both correctness and idiomatic fit, revised based on that feedback, and
shipped as an update to the existing marketplace plugin.
