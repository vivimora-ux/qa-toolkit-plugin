# Testing guide: `rs-aut`, `pr-explainer`, `test-plan`, and `test-suite`

For QA testers verifying the `/rs-aut`, `/pr-explainer`, `/test-plan`, and
`/test-suite` skills, and their six shared modifier arguments, on a project
of your choice.

## Prerequisites

- Pick a target project (not this plugin repo) to test in. It should
  have real git history and at least one real PR, or an unmerged local
  branch with changes — the `pr-explainer` tests need something to diff
  against.
- In that project, install the plugin from inside Claude Code:

```
/plugin marketplace add vivimora-ux/aut-onboarding-marketplace
/plugin install qa-toolkit@aut-onboarding-marketplace
```

## What you're testing

- **4 skills**: `/rs-aut [modifiers] [topic]`,
  `/pr-explainer [modifiers] [PR#|URL|blank]`,
  `/test-plan [modifiers] [PR#|URL|feature area|blank]`,
  `/test-suite [modifiers]`
- **6 modifier arguments**, shared across all four skills, in effect for
  the rest of the session once set:
  - Skill level: `junior`, `mid`, `senior`
  - Entry point: `visual`, `trace`, `risk`

Replace `<PR#>`, `<PR-URL>`, and `<topic>` below with real values from
your target project.

## Test 1 — Defaults (no modifiers set)

```
/rs-aut help me understand this project
```

Expect: mid-level depth, a balanced pass (structure → trace → risk),
and a brief mention that other levels exist. Check that
`docs/onboarding/rs-aut_<today's date>.md` is created in the
target project, with a header block (`Created:`, `Commands used:`) and
real `##` headers.

## Test 2 — Each modifier individually, on `/rs-aut`

Run these one at a time, in a fresh session each, to confirm each
behaves distinctly:

```
/rs-aut junior the main data flow
```

```
/rs-aut senior the main data flow
```

```
/rs-aut visual architecture
```

```
/rs-aut trace the main data flow
```

```
/rs-aut risk testing strategy
```

Expect visibly different tone/framing per modifier, not just a label
change:
- `junior` defines terms and ends with a confirmation-style check-in.
- `senior` skips definitions and goes straight to edge cases.
- `visual` leads with architecture.
- `trace` leads with a step-by-step walk-through.
- `risk` leads with fragility/coverage gaps.

## Test 3 — Combining modifiers, one session

```
/rs-aut senior risk <topic>
```

Expect senior vocabulary and risk-first framing combined in the same
answer.

## Test 4 — Modifiers persist and carry over to `/pr-explainer`

In the *same* session as Test 3 (don't reset), without resetting the
modifiers, switch skills:

```
/pr-explainer
```

Expect it still applies `senior`/`risk` from earlier — risk-led, no
re-explaining terms — confirming that modifiers are shared across both
skills.

## Test 5 — `/pr-explainer` input sources

Run each in its own session (reset modifiers with `/rs-aut mid` first
if you want a clean baseline):

- `/pr-explainer <PR#>` — should use `gh pr view` / `gh pr diff`.
- `/pr-explainer <PR-URL>` — same, via URL.
- `/pr-explainer` with no argument, on a branch with local changes —
  should diff against the default branch.
- `/pr-explainer` with no argument, on a clean branch with nothing to
  compare — should say plainly there's nothing to review, not invent
  content.
- (Optional) simulate `gh` not being authenticated (`gh auth logout`
  temporarily, only in a throwaway/safe environment) — should say so
  plainly and fall back to the local diff instead of failing silently.

## Test 6 — The seven-topic coverage in `/pr-explainer`

On one real PR, check the answer actually hits: change summary, scope
of impact, code walk-through, test coverage (explicitly flagging logic
changes with no accompanying test changes), risk assessment, suggested
test plan (priority-ordered), and non-functional considerations
(skipped briefly if not applicable, not omitted silently).

## Test 7 — File output correctness

After a few runs on the same day:

- Confirm `docs/onboarding/rs-aut_<today's date>.md` and
  `docs/pr-explainer/pr-explainer_pr<PR#>_<today's date>.md` (or
  `pr-explainer_<branch>_<today's date>.md`) exist in the *target*
  project, not this plugin repo.
- Run a second `/rs-aut` or `/pr-explainer` later the same day on
  the same topic/PR — confirm it **updates** the existing file
  (appends to "Commands used", updates matching sections) rather than
  creating a duplicate file.
- Confirm the "Commands used" list accumulates every modifier and
  invocation typed so far, in order, as inline code.

## Test 8 — `/test-plan` scope detection and missing-doc handling

Run each in its own session, against the target project:

- `/test-plan <PR#>` (with a matching `docs/pr-explainer/` doc already
  on disk from Test 5/6) — should read that doc and produce a
  PR-scoped plan, without re-reading the diff itself.
- `/test-plan <feature area>` (with a matching `docs/onboarding/` doc
  already on disk from Test 1) — should read that doc and produce a
  project-scoped plan.
- `/test-plan <PR#>` where **no** matching `docs/pr-explainer/` doc
  exists yet — should say so plainly and suggest running
  `/pr-explainer` first, not analyze the diff itself to fill the gap.
- `/test-plan <feature area>` where **no** `docs/onboarding/` doc
  exists yet — should say so plainly and suggest `/rs-aut` first.

## Test 9 — Modifiers carry over to `/test-plan`, and tech suggestions

In the same session as Test 3/4 (don't reset), without resetting the
modifiers:

```
/test-plan <PR#>
```

Expect it still applies `senior`/`risk` from earlier, confirming
modifiers are shared across all three skills. Also confirm:

- Automated test case suggestions name a specific tool the `rs-aut` doc's
  **Technology stack** section already listed for that layer (e.g.
  Playwright already in place → suggests Playwright, not Cypress),
  and only suggest something different when the doc gave an explicit
  reason (no tooling yet, or tooling flagged as technical debt).
- If this run had no `rs-aut` doc to draw from (PR-scoped only), it
  says so and skips technology suggestions rather than guessing the
  stack directly.

## Test 10 — `/test-plan` file output

After a run or two on the same day:

- Confirm `docs/test-plan/test-plan_pr<PR#>_<today's date>.md` (or
  `test-plan_<today's date>.md` for a project-scoped run) exists in the
  *target* project, with a header block (`Created:`, source doc
  path(s), `Commands used:`).
- Run a second `/test-plan` later the same day on the same scope —
  confirm it **updates** the existing file rather than duplicating it.

## Test 11 — `/test-suite` with no `rs-aut` docs

In a fresh target project (or one with `docs/onboarding/` temporarily
renamed aside), run:

```
/test-suite
```

Expect it to say plainly that no `rs-aut` docs exist and suggest running
`/rs-aut` first — it should not analyze the project itself to fill the
gap.

## Test 12 — `/test-suite` multi-doc reading and coverage gaps

With at least two `docs/onboarding/rs-aut_*.md` files on disk for
different areas of the project (from earlier `/rs-aut` runs), run:

```
/test-suite
```

Expect:

- The inventory draws on *all* onboarding docs found, not just the most
  recent one.
- A gap-disclosure note near the top, naming any of the seven `rs-aut`
  topics (or feature areas) with no onboarding doc yet.
- Every case tagged with both a priority (highest-risk first, traced to
  something an `rs-aut` doc actually flagged) and a type (functional /
  edge / negative / integration).

## Test 13 — Modifiers carry over to `/test-suite`

In the same session as Test 3/4/9 (don't reset), without resetting the
modifiers:

```
/test-suite
```

Expect it still applies `senior`/`risk` from earlier, confirming
modifiers are shared across all four skills. Also check the
`visual`/`trace` variants specifically:

- `/test-suite visual` — groups the inventory by module/component first.
- `/test-suite trace` — orders cases along a real user/data flow,
  end to end, rather than by module.

## Test 14 — `/test-suite` file output

After a run or two on the same day:

- Confirm `docs/test-suite/test-suite_<today's date>.md` exists in the
  *target* project, with a header block (`Created:`, the source `rs-aut`
  doc path(s) actually read, `Commands used:`, and a `Coverage gaps` note).
- Run a second `/test-suite` later the same day — confirm it **updates**
  the existing file rather than duplicating it.

## Quick pass/fail checklist

- [ ] Plugin installs cleanly in a separate project
- [ ] Each of the 6 modifiers produces visibly distinct behavior when
      passed as an argument
- [ ] Modifiers persist across the session and across all four skills
- [ ] `/pr-explainer` handles: PR number, PR URL, blank/local diff,
      no-`gh`, nothing-to-review
- [ ] `/test-plan` correctly detects PR-scoped vs. project-scoped runs,
      and refuses to guess when the source doc is missing
- [ ] `/test-plan` technology suggestions match what `rs-aut` found in
      place, deviating only with a stated reason
- [ ] `/test-suite` refuses to invent coverage when no `rs-aut` doc
      exists, and discloses gaps plainly when docs are only partial
- [ ] `/test-suite` correctly combines multiple `rs-aut` docs, and every
      case carries both a priority and a type
- [ ] Files are written to the target project's `docs/onboarding/`,
      `docs/pr-explainer/`, `docs/test-plan/`, and `docs/test-suite/`,
      not this repo
- [ ] Same-day reruns update files in place instead of duplicating
