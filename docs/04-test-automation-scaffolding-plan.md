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

- **Framework choice is explicit and persists, at the project level, not
  the session level.** The person sets it once (as an argument, or
  it's defaulted from `rs-aut`'s detected tech stack) and it carries
  over for the project. No new command layer — resolved via file-based
  state instead; see "Open questions / risks" below.
- **This does not replace `test-suite`.** `test-automate` only ever reads
  a `test-suite` doc (or a `test-plan` doc, for a narrower PR/area-scoped
  automation pass) — it doesn't invent test cases itself.
- **Scope for v1: new/greenfield automation only.** This phase targets
  projects with little or no existing automated tests. Retrofitting or
  refactoring an already-substantial existing test suite is out of scope
  for now (see Non-goals).
- **Skill entry point, Agent fan-out for generation.** `test-automate`
  is a Skill (consistent with the rest of the plugin, and because
  comment-density modifiers and framework-choice resolution need
  main-conversation/session continuity), but the actual per-case code
  generation is delegated to one Agent per target spec file, not
  written inline — generated code volume and the naturally
  parallelizable, file-independent shape of the work make this a
  better fit than the narrative-style Skills elsewhere in the plugin.
  Cases are grouped by target spec file before fan-out so no two
  agents write the same file.

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
├── reference/
│   ├── modifiers.md         # updated — notes test-automate's
│   │                        #  skill-level-only modifier scope
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
   Implemented as an Agent fan-out: cases are grouped by target spec
   file first, then one Agent per target file generates and writes that
   file's code in parallel — not generated inline by the Skill itself.
4. **Report what was skipped.** Any case marked manual-only in the
   source doc is listed as skipped, with the reason, rather than
   silently omitted.

### Session/project modifiers

- `/junior` / `/mid` / `/senior` — control comment density and
  explanatory depth in generated code, same mechanism as elsewhere.
  `/visual` / `/trace` / `/risk` don't apply to this skill — there's no
  equivalent framing concept for writing code files.
- Framework choice — persists at the project level (not just the
  session), since switching frameworks mid-project is a much bigger
  decision than switching explanation depth. Resolved via the
  file-based state mechanism in "Open questions / risks" below.

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

- [x] `test-automate/SKILL.md` drafted, correctly reading either a
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

- **Framework-choice persistence — resolved.** No new commands and no
  new config-file format. `test-automate` resolves the framework in
  order: an explicit argument (`/test-automate playwright`) > the most
  recent `docs/test-automate/test-automate_*.md` file's `Framework:`
  header line from a prior run > whatever `rs-aut`'s "Technology stack"
  section already found in place > asking once if none of those apply.
  This reuses the same dated-markdown-with-header-block mechanism every
  other skill already uses for state, rather than inventing a new one.
  See `qa-toolkit-plugin/skills/test-automate/SKILL.md` → "Resolving the
  framework."
- **Manual vs. automatable tagging gap — resolved.** `test-plan` docs
  already separate "Manual test cases" from "Automated test case
  suggestions," but `test-suite` docs only tagged `priority` and `type`
  — no automatable/manual signal. Fixed with a small, additive
  `automatable: yes`/`no` tag added to `test-suite`'s case tagging (see
  `qa-toolkit-plugin/skills/test-suite/SKILL.md` → "Automatable
  tagging"), so `test-automate` never has to guess or infer
  automatability itself.
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
- **Parallel-write conflicts.** If two test cases map to the same
  existing spec file, generating them via separate concurrent Agents
  without first grouping by target file could corrupt or drop one
  write. Mitigation: group cases by target file before fan-out (one
  Agent per file, never two Agents per file) — needs to be built into
  the implementation, not left as a runtime assumption.

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
