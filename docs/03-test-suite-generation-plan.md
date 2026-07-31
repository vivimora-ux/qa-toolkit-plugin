# [Story] Full test-suite generation skill (Phase 3)

**Type:** Story (part of the "Personalized schema for accelerating AUT
understanding & judgment" epic)
**Priority:** Medium-High
**Reporter:** _(unassigned — draft, no ticket number yet)_
**Team:** QA / QE
**Labels:** `ai-tooling`, `qa-enablement`, `claude-code`, `test-automation`

---

## Summary

Add a new Skill, `test-suite`, to the existing `qa-toolkit` plugin that
generates a full, exhaustive test case inventory for a project — every
manual and automatable test case a QA should have written, covering the
whole project rather than one PR or one feature area. This is the
project-wide counterpart to `test-plan`: `test-plan` is deliberately
narrow and reads a single `pr-explainer` or `rs-aut` doc to produce a
scoped, prioritized plan for one change or area; `test-suite` reads across
all of a project's `rs-aut` docs to produce comprehensive coverage for
projects that currently have little or no test automation in place.

This is Phase 3, framework-agnostic. Phase 4 (a separate story) takes
this skill's output and a chosen automation framework (Playwright,
WebdriverIO, etc.) and generates actual test code from it.

## Background

The team has projects with little or no existing test automation. For
those, there's no PR or feature area to scope a plan against yet — what's
needed first is a from-scratch inventory: what *should* be tested, across
the whole application, before any automation work starts. `test-plan`
can't do this today by design — it explicitly refuses to analyze
anything itself and only reads what `pr-explainer`/`rs-aut` already
found for a specific PR or area. `test-suite` fills that gap: it reads
broadly across a project's existing `rs-aut` docs (ideally one per major
feature/area, run over time) to build the full inventory.

## Decisions made so far

- **This is a new Skill, not a mode of `test-plan`.** `test-plan` stays
  scoped and secondary by design (single PR/area, reads-only). `test-suite`
  is intentionally broader — project-wide, not tied to one change — so it
  gets its own Skill rather than overloading `test-plan`'s narrower
  contract.
- **Framework-agnostic in this phase.** Output is test cases in plain
  language plus structural metadata (module, priority, type), not code.
  Framework/tool choice and code generation are explicitly deferred to
  Phase 4.
- **Reuses the shared modifiers.** Same `/junior`, `/mid`, `/senior`,
  `/visual`, `/trace`, `/risk` commands from `rs-aut` — no new modifiers
  introduced in this phase.

## Goals

- New Skill: `test-suite`, added to the existing `qa-toolkit` plugin
  alongside `rs-aut`, `pr-explainer`, and `test-plan`.
- Reads all available `docs/onboarding/rs-aut_*.md` files for the project
  (not just the most recent one) to assemble as complete a picture of
  testable surface area as the existing onboarding docs allow.
- If onboarding docs don't yet cover the whole project, says so plainly
  and names which areas are missing, rather than inventing coverage for
  parts of the project nobody has run `/rs-aut` against yet.
- Produces a full test case inventory, organized by feature/module,
  covering:
  1. Functional cases (expected/happy-path behavior)
  2. Edge cases and boundary conditions
  3. Negative/error-handling cases
  4. Integration points between modules or with external systems
     (e.g. Data Cloud integrations, flow automation, if `rs-aut`'s docs
     found them)
- Each case tagged with a priority (highest-risk first, using the same
  risk signals `rs-aut`'s docs already surfaced — fragile areas, past
  breakage, technical debt) and a type (functional / edge / negative /
  integration).
- Reuses `/junior`, `/mid`, `/senior` for how much rationale accompanies
  each case, and `/visual` / `/trace` / `/risk` for how the inventory is
  organized — same mechanism as `test-plan`, same modifiers, same
  session-persistence behavior.
- Same file-writing behavior as the other three skills: automatically
  saved, dated, updated in place on same-day reruns, with a header block
  listing commands used.

## Non-goals

- Not generating any test code or picking/assuming an automation
  framework — that's Phase 4, once a framework is chosen.
- Not reading or re-analyzing PR diffs — this skill is project-wide and
  point-in-time, not change-scoped. `pr-explainer`/`test-plan` already
  cover the PR-specific case.
- Not replacing `test-plan` — for a single PR or feature area,
  `test-plan` remains the right, narrower tool. `test-suite` is for
  projects (or areas of a project) with no prior test coverage inventory
  at all.
- Not deciding test priority/order in a way that overrides `risk`-flagged
  items from `rs-aut` — priority here should trace back to what the
  onboarding docs actually found, not a fresh risk assessment invented by
  this skill.
- Not auto-triggering — explicit `/test-suite` invocation only, same as
  the other three skills.

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
│   └── test-suite/
│       └── SKILL.md         # NEW
├── reference/
│   └── modifiers.md         # unchanged — shared by all four skills now
└── README.md                # updated with /test-suite usage
```

### The Skill (`test-suite`)

When invoked:

1. **Gather source material.** Look for every `docs/onboarding/rs-aut_*.md`
   file in the project (not just today's). If the project has been
   onboarded piecemeal (different areas on different days), combine all
   of them. If no `rs-aut` docs exist at all, say so plainly and suggest
   running `/rs-aut` first — don't analyze the project from scratch to
   fill the gap, same rule `test-plan` already follows for its own
   sources.
2. **Identify coverage gaps.** Compare what the onboarding docs actually
   cover against the six standard topics. Name explicitly which
   feature areas or topics have no onboarding doc yet, so the person
   knows the inventory is partial where it's partial.
3. **Build the inventory.** For each covered area, derive functional,
   edge, negative, and integration cases strictly from what the
   `rs-aut` doc(s) actually describe (architecture, data flow, flow
   automation requirements, testing strategy) — never invent behavior
   the docs didn't describe.
4. **Prioritize.** Highest-risk first, using whatever `rs-aut` already
   flagged (fragile dependencies, past breakage, technical debt) as the
   basis — this skill doesn't run its own independent risk analysis.

### Session modifiers

Shared with the other three skills, per `reference/modifiers.md`:
- `visual` — group the inventory by module/component first.
- `trace` — order cases along real user/data flows end to end.
- `risk` — this is the default ordering; under this modifier, also state
  briefly why each top area is highest-risk (same pattern as `test-plan`).
- `junior` / `mid` / `senior` — control how much rationale accompanies
  each case, same behavior as elsewhere.

### File output

- **Location** — `docs/test-suite/` at the root of the project being
  tested.
- **Filename** — `test-suite_<YYYY-MM-DD>.md`. One file per day,
  updated in place on reruns the same day, consistent with `rs-aut` and
  `test-plan`'s conventions.
- **Header block** — creation date, list of source `rs-aut` doc paths
  actually read, and a "Commands used" section (same format as the other
  three skills).
- **Body** — the inventory organized under clear markdown headers,
  grouped by module (or by flow, under `visual`/`trace`), each case
  tagged with priority and type.

## Acceptance criteria

- [ ] `test-suite/SKILL.md` drafted, producing functional, edge,
      negative, and integration cases
- [ ] Correctly reads and combines multiple `rs-aut` docs when more than
      one exists for a project
- [ ] Explicitly names any topic/feature area with no onboarding doc yet,
      rather than silently omitting or inventing it
- [ ] No-source-doc case (`/test-suite` with zero `rs-aut` docs present)
      says so plainly and suggests `/rs-aut` first, without analyzing
      anything itself
- [ ] `/junior`, `/mid`, `/senior` produce meaningfully different
      rationale depth, consistent with the other three skills
- [ ] `/visual`, `/trace`, `/risk` produce genuinely different
      organization of the same inventory
- [ ] File output follows the same header/update/no-duplicate
      conventions as `rs-aut`, `pr-explainer`, and `test-plan`
- [ ] Piloted on at least one real project that currently has little or
      no automation, reviewed by a senior for completeness (nothing
      obviously testable left out) and non-invention (nothing claimed
      that the source docs didn't actually support)
- [ ] Plugin version bumped and pushed; a teammate confirms
      `/plugin marketplace update` picks up the new skill

## Open questions / risks

- **Coverage depends entirely on how thorough the underlying `rs-aut`
  docs are.** If `rs-aut` was only run against a couple of features, the
  inventory will reflect that partial picture. Worth deciding whether
  `test-suite` should nudge the person toward running `/rs-aut` against
  remaining areas before treating an inventory as "done," or just always
  disclose the gap and leave the call to the person.
- **Inventory size on larger projects.** A truly whole-project case list
  could get long. Worth watching during pilot whether it needs
  pagination/splitting by module into separate files, versus one long
  document.
- **Where this sits relative to `test-plan` in practice.** Once
  `test-suite` exists, team members need a clear mental model for when
  to reach for which: `test-suite` for "we have nothing yet, what's the
  full list," `test-plan` for "this PR/area just changed, what do I test
  now." Worth stating this distinction clearly in the README so it isn't
  confused with `test-plan`.

## Suggested subtasks

1. Draft `test-suite/SKILL.md`
2. Test multi-doc reading against a project with more than one `rs-aut`
   doc on disk
3. Test the no-source-doc and partial-coverage cases explicitly
4. Internal review with 2+ seniors on a real low-automation project —
   check for completeness and non-invention
5. Revise based on review feedback
6. Bump plugin version, update README (including the `test-suite` vs.
   `test-plan` distinction), push to marketplace repo
7. Confirm update path works for a teammate via `/plugin marketplace
   update`

## Definition of done

`test-suite` is drafted, reads across all available `rs-aut` docs for a
project, produces a prioritized functional/edge/negative/integration
test case inventory with honest gap disclosure, has been reviewed by
senior team members against a real low-automation project, revised
based on that feedback, and shipped as an update to the existing
marketplace plugin — with the README updated to make its relationship to
`test-plan` clear.
