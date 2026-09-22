# Testing guide: `rs-aut`, `pr-explainer`, `test-plan`, `test-suite`, and `test-automate`

For QA testers verifying the `/rs-aut`, `/pr-explainer`, `/test-plan`,
`/test-suite`, and `/test-automate` skills, on a project of your choice.
The first four share six modifier arguments; `test-automate` only uses
three of them (see below).

## Prerequisites

- Pick a target project (not this plugin repo) to test in. It should
  have real git history and at least one real PR, or an unmerged local
  branch with changes — the `pr-explainer` tests need something to diff
  against.
- In that project, install the plugin from inside Claude Code:

```
/plugin marketplace add vivimora-ux/qa-toolkit-plugin
/plugin install qa-toolkit@qa-toolkit-plugin
```

## What you're testing

- **5 skills**: `/rs-aut [modifiers] [topic]`,
  `/pr-explainer [modifiers] [PR#|URL|blank] [github]`,
  `/test-plan [modifiers] [feature area|blank]`,
  `/test-suite [modifiers] [jira [PROJECT-KEY]]`,
  `/test-automate [junior|mid|senior] [framework] [feature area|blank]`
- **6 modifier arguments**, shared across the first four skills, in effect
  for the rest of the session once set. `test-automate` only uses skill
  level (depth), never entry point (framing) — see Test 24:
  - Skill level: `junior`, `mid`, `senior`
  - Entry point: `visual`, `trace`, `risk`
- `pr-explainer` also has a separate, optional GitHub MCP enrichment —
  not part of the modifier system above, off by default — covered in
  Tests 8-9.
- `test-suite` also has a separate, optional Jira integration — not part
  of the modifier system above, off by default — covered in Tests 17-20.

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

## Test 8 — Enabling GitHub MCP enrichment and the first-time auth prompt

In a target project with at least one real PR to review, and no prior
GitHub MCP configuration, run:

```
/pr-explainer <PR#> github
```

Expect:

- It says plainly that it's connecting to GitHub via the MCP
  integration, walking through authenticate → present URL →
  complete-authentication if the real GitHub MCP tools aren't available
  yet (this may only need to happen once across your test session).
- Once connected, no further question is asked — there's no key or
  project to configure here, unlike `test-suite`'s Jira integration.
- The file's header now records a `GitHub MCP: enabled` line.
- Run a plain `/pr-explainer <other-PR#>` (no `github` argument, no
  persisted enablement, a different project) — confirm no GitHub MCP
  prompt or connection attempt happens at all. GitHub MCP must never be
  engaged unless `github` was actually passed or is already persisted
  for that project.

## Test 9 — GitHub MCP persistence, enrichment content, and non-PR scope skip

Later the same day (or in a new session against the same project), run:

```
/pr-explainer <PR#>
```

Expect it reuses the persisted `GitHub MCP: enabled` state from Test 8
without asking or requiring the `github` argument again, and that:

- **Risk assessment** incorporates existing review comments on the PR
  without re-flagging a concern a reviewer already raised — it builds on
  that, doesn't duplicate it.
- The **non-functional considerations** or risk section reflects the
  PR's current CI/check and approval status.
- **Change summary** reflects any linked issue's stated intent, the same
  way a PR description already does.
- All of the above is treated exactly like `gh`-sourced content: already
  decided, never expanded on or second-guessed.

Then, with GitHub MCP still enabled, run against a local branch diff
(no PR number/URL):

```
/pr-explainer
```

Expect it skips GitHub MCP enrichment for this run — there's no PR
entity to enrich — without failing, erring, or asking anything.
Confirm the rest of the review (from the local diff) still runs
normally.

## Test 10 — `/test-plan` scope detection and missing-doc handling

Run each in its own session, against the target project:

- `/test-plan <feature area>` (with a matching `docs/onboarding/` doc
  already on disk from Test 1) — should read that doc and produce an
  area-scoped plan, without re-exploring the project itself.
- `/test-plan` with no argument (with a `docs/onboarding/` doc already on
  disk) — should read the most recent doc and produce a whole-project-
  scoped plan.
- `/test-plan <feature area>` where **no** `docs/onboarding/` doc
  exists yet — should say so plainly and suggest `/rs-aut` first, not
  analyze the project itself to fill the gap.

## Test 11 — Modifiers carry over to `/test-plan`, and tech suggestions

In the same session as Test 3/4 (don't reset), without resetting the
modifiers:

```
/test-plan <feature area>
```

Expect it still applies `senior`/`risk` from earlier, confirming
modifiers are shared across all three skills. Also confirm:

- Automated test case suggestions name a specific tool the `rs-aut` doc's
  **Technology stack** section already listed for that layer (e.g.
  Playwright already in place → suggests Playwright, not Cypress),
  and only suggest something different when the doc gave an explicit
  reason (no tooling yet, or tooling flagged as technical debt).

## Test 12 — `/test-plan` file output

After a run or two on the same day:

- Confirm `docs/test-plan/test-plan_<today's date>.md` exists in the
  *target* project, with a header block (`Created:`, source `rs-aut` doc
  path(s), `Commands used:`).
- Run a second `/test-plan` later the same day on the same scope —
  confirm it **updates** the existing file rather than duplicating it.

## Test 13 — `/test-suite` with no `rs-aut` docs

In a fresh target project (or one with `docs/onboarding/` temporarily
renamed aside), run:

```
/test-suite
```

Expect it to say plainly that no `rs-aut` docs exist and suggest running
`/rs-aut` first — it should not analyze the project itself to fill the
gap.

## Test 14 — `/test-suite` multi-doc reading and coverage gaps

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
- Each module has a summary table with `ID | Title | Priority | Type |
  Automatable | Status` columns — priority highest-risk-first, traced to
  something an `rs-aut` doc actually flagged, type one of functional /
  edge / negative / integration.
- Below each summary table, a `####` detail block per case (matching its
  ID) with Objective, Preconditions, Test data, numbered Steps, and
  Expected result — plus blank/`Not Run` Actual result, Status, and
  Comments placeholders.

## Test 15 — Modifiers carry over to `/test-suite`

In the same session as Test 3/4/11 (don't reset), without resetting the
modifiers:

```
/test-suite
```

Expect it still applies `senior`/`risk` from earlier, confirming
modifiers are shared across all four skills. Also check the
`visual`/`trace` variants specifically:

- `/test-suite visual` — groups the inventory by module/component first.
- `/test-suite trace` — orders cases along a real user/data flow, end to
  end, rather than by module — confirm the same summary-table-plus-
  detail-block shape is used, just grouped by flow (e.g. "### Checkout
  flow") instead of by module.

## Test 16 — `/test-suite` file output

After a run or two on the same day:

- Confirm `docs/test-suite/test-suite_<today's date>.md` exists in the
  *target* project, with a header block (`Created:`, the source `rs-aut`
  doc path(s) actually read, `Commands used:`, and a `Coverage gaps` note).
- Run a second `/test-suite` later the same day — confirm it **updates**
  the existing file rather than duplicating it, and that every case's ID
  stays the same as before (no renumbering).
- Manually edit one case's Status/Actual result/Comments cells in the
  file (as if a tester ran it), then run `/test-suite` again the same
  day on an unrelated area — confirm your manual edits are still there
  afterward, untouched.

## Test 17 — Enabling Jira integration and the first-time key prompt

In a project with at least one `rs-aut` doc already on disk (from Test
13/14), and no prior Jira configuration, run:

```
/test-suite jira
```

Expect:

- It says plainly that it's connecting to Jira via the Atlassian Rovo
  integration, walking through authenticate → present URL →
  complete-authentication if the real Jira tools aren't available yet
  (this may only need to happen once across your test session).
- Once connected, it asks once for a Jira project or epic key — it does
  not guess one.
- After you answer, the file's header (see Test 20) records a `Jira
  project:` line with the key you gave.
- Run a plain `/test-suite` (no `jira` argument, no persisted key) in a
  different project — confirm no Jira prompt or connection attempt
  happens at all. Jira must never be engaged unless `jira` was actually
  passed or a key is already persisted.

## Test 18 — Jira key persistence and override

Later the same day (or in a new session against the same project), run:

```
/test-suite
```

Expect it reuses the persisted `Jira project:` key from Test 17 without
asking again, and continues reading Jira automatically. Then run:

```
/test-suite jira OTHERPROJ
```

Expect the explicit key wins and re-persists — confirm a subsequent
plain `/test-suite` uses `OTHERPROJ`, not the original key.

## Test 19 — Combined rs-aut + Jira inventory and extended coverage-gap disclosure

With Jira integration configured (Test 17/18) and pointed at a project
or epic with at least one issue whose requirements/AC cover an area also
covered by an `rs-aut` doc, and at least one area where only one side
has coverage, run:

```
/test-suite
```

Expect:

- The gap-disclosure note near the top also names any module with
  `rs-aut` coverage but no matching Jira requirement, and any Jira
  requirement with no matching `rs-aut` architecture context.
- For the area with both sources, cases draw on `rs-aut` for
  architecture/data-flow facts and Jira for functional
  requirements/acceptance criteria, without expanding on or
  second-guessing either source's actual text.

## Test 20 — Opt-in Jira write: single tracking issue, never automatic

First confirm that plain `/test-suite` runs (even with Jira configured)
never create or update anything in Jira on their own — check the Jira
project for the absence of any new issue after Tests 17-19. Then
explicitly ask, in the same session, in your own words (this is loosely
detected, not a fixed required phrase) something like:

```
push this test-suite inventory to Jira
```

Expect:

- A single summary tracking issue is created in the configured project
  — not one per test case — containing case counts/breakdown and a link
  back to the local `docs/test-suite/test-suite_<today's date>.md` file.
- The local file's header now records a `Jira tracking issue:` line.
- Ask again later the same day — confirm it updates the same tracking
  issue in place rather than duplicating it, matching the local file's
  one-per-day convention.

## Test 21 — `/test-automate` with no source doc

In a project with no `docs/test-suite/` or `docs/test-plan/` doc, run:

```
/test-automate
```

Expect it to say so plainly and suggest `/test-suite` or `/test-plan`
first — it should not analyze the project or generate any code itself
to fill the gap.

## Test 22 — Framework detection, persistence, and override

With a `docs/test-suite/` or `docs/test-plan/` doc present, and no prior
`/test-automate` run yet:

```
/test-automate
```

Expect: if `rs-aut`'s doc already found a framework in place, it's used
without asking; otherwise it asks once. Then run again the same day:

```
/test-automate
```

Expect the same framework reused without asking again (read from the
prior run's summary doc header). Then explicitly override:

```
/test-automate webdriverio
```

Expect the override to take effect immediately and persist for
subsequent runs.

## Test 23 — Automatable filtering and skipped-case reporting

- Against a `docs/test-suite/` doc: confirm only cases with
  `Automatable: Yes` are turned into specs, and every `Automatable: No`
  case appears in the skipped-cases report with its reason.
- Against a `docs/test-plan/` doc: confirm only cases under "Automated
  test case suggestions" are turned into specs, and cases that only
  appear under "Manual test cases" are reported as skipped, not
  force-converted.
- If a generation agent can't sensibly automate a case it was assigned
  (a generation-time skip), confirm it's reported too, and that the
  final report doesn't silently drop either skip source.

## Test 24 — Generated file correctness and modifier scope

- Open a generated spec file and confirm it has a traceability comment
  linking back to its source case and source doc path.
- Compare the same case generated under `/test-automate junior` vs.
  `/test-automate senior` — expect visibly different comment density
  (junior: explains most assertions; senior: comment-light, only the
  traceability comment).
- If the source doc has two cases that map to the same target spec
  file, confirm they land in **one** file with both tests present, not
  two separate/conflicting files.
- Confirm `/test-automate visual` or `/test-automate risk` either has
  no effect or is declined — `test-automate` doesn't use entry-point
  modifiers, unlike the other four skills.

## Quick pass/fail checklist

- [ ] Plugin installs cleanly in a separate project
- [ ] Each of the 6 modifiers produces visibly distinct behavior when
      passed as an argument
- [ ] Modifiers persist across the session and across all four skills
- [ ] `/pr-explainer` handles: PR number, PR URL, blank/local diff,
      no-`gh`, nothing-to-review
- [ ] `/pr-explainer` never engages GitHub MCP (including
      authentication) without the `github` argument or a persisted
      enablement, and skips enrichment gracefully on a local-diff (no
      PR) run
- [ ] `/pr-explainer`'s GitHub MCP enrichment (review comments, CI/
      approval status, linked issues) is additive to `gh`-sourced
      content, never a replacement, and never expanded on or
      second-guessed
- [ ] `/test-plan` correctly detects feature-area vs. whole-project runs,
      and refuses to guess when the source `rs-aut` doc is missing
- [ ] `/test-plan` technology suggestions match what `rs-aut` found in
      place, deviating only with a stated reason
- [ ] `/test-suite` refuses to invent coverage when no `rs-aut` doc
      exists, and discloses gaps plainly when docs are only partial
- [ ] `/test-suite` correctly combines multiple `rs-aut` docs, and every
      case has a stable ID, a summary-table row (priority, type,
      automatable, status), and a matching detail block
- [ ] `/test-suite` never engages Jira (including authentication) on a
      plain run with no `jira` argument and no persisted key
- [ ] `/test-suite` writing to Jira only happens on an explicit ask,
      creates exactly one tracking issue per project (never per-case),
      and updates that same issue in place on same-day reruns
- [ ] `/test-automate` refuses to generate anything when no source doc
      exists, and never invents a test case not in that doc
- [ ] `/test-automate` framework resolution follows the right order:
      explicit argument > persisted choice > `rs-aut` detection > ask once
- [ ] `/test-automate` correctly filters to automatable/automated-suggested
      cases only, and reports every skip (pre-filtered and generation-time)
- [ ] `/test-automate` groups cases by target spec file before generating,
      so two cases for the same file never produce conflicting writes
- [ ] `/test-automate` comment density visibly differs across
      `junior`/`mid`/`senior`, and ignores `visual`/`trace`/`risk`
- [ ] Files are written to the target project's `docs/onboarding/`,
      `docs/pr-explainer/`, `docs/test-plan/`, `docs/test-suite/`, and
      `docs/test-automate/` (plus real spec files in the project's test
      directory for `test-automate`), not this repo
- [ ] Same-day reruns update files in place instead of duplicating
