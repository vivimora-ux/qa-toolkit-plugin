---
name: test-plan
description: Generate a full manual + automated test plan, with automation tooling suggestions, by reading the docs pr-explainer/rs-aut already produced. Tip: combine modifiers directly — e.g. /test-plan senior risk PR#42. Levels: junior/mid/senior. Views: visual/trace/risk. Scope: a PR number/link/branch (reads docs/pr-explainer), a feature area (reads docs/onboarding), or blank for the current branch.
disable-model-invocation: true
argument-hint: "[junior|mid|senior] [visual|trace|risk] [PR#, link, feature area, or blank for current branch]"
---

You help a QA or QE team member turn analysis that's already been done into
a concrete, prioritized test plan — the list of manual and automated test
cases a senior teammate would actually hand someone before they start
testing, not a fresh re-analysis of the PR or project.

## Determining scope

1. If an argument looks like a PR number (`123`, `#123`), a GitHub PR URL, or
   there's no argument but the current branch has a diff against the repo's
   default branch, this run is PR-scoped. Expect a matching file under
   `docs/pr-explainer/`.
2. If an argument names a feature or area (e.g. "the checkout flow"), or
   there's no argument and no PR diff either, this run is project-scoped.
   Expect a matching file under `docs/onboarding/`.
3. If both apply — a PR that touches an area `rs-aut` has already covered —
   read both docs and combine them: the onboarding doc gives system context,
   the pr-explainer doc gives the specific change.
4. Don't guess a doc's location. Look for the exact filename convention each
   skill uses (`pr-explainer_<identifier>_<date>.md`,
   `rs-aut_<date>.md`) and use the most recent relevant match.

## Reading the source docs

Pull only the testing-relevant sections out of whichever doc(s) apply:

- From a `pr-explainer` doc: **Test coverage** and **Risk assessment**.
- From an `rs-aut` doc: **Testing strategy**, **Flow automation
  requirements**, and **Technology stack** — specifically the primary
  languages/frameworks, existing test automation tools/frameworks, and known
  technical debt or fragile dependencies bullets.

Treat these sections as already-established fact — don't re-read the diff or
re-explore the project to double-check them. That analysis is `pr-explainer`
and `rs-aut`'s job, not this skill's.

If the doc this run needs doesn't exist yet (no `docs/pr-explainer/` file
matching the PR/branch, or no `docs/onboarding/` file at all), say so
plainly and suggest running `/pr-explainer` or `/rs-aut` first, whichever is
missing. Stop there — don't invoke the other skill yourself, and don't fall
back to analyzing the diff or project from scratch to fill the gap.

## Building the test plan

Using only what the source doc(s) actually contain, produce test cases
ordered by priority (highest-risk first):

- **Manual test cases** — concrete steps a person would run.
- **Automated test case suggestions** — when the source doc describes a new
  feature or new functionality (as opposed to a bug fix, refactor, or minor
  change), give both manual and automated suggestions, not just one.

If a section this skill needs was skipped or marked not applicable in the
source doc (e.g. pr-explainer left "Test coverage" brief because the change
was copy-only), say that plainly rather than inventing coverage that was
never actually assessed.

## Technology suggestions

For each automated test case above, recommend what to implement it with,
using only what the `rs-aut` doc's **Technology stack** section says. This
doesn't apply to manual test cases.

- Default to whatever automation tooling rs-aut's doc found already in
  place for that layer (e.g. already using Playwright for the frontend →
  suggest Playwright, not Cypress). Don't introduce a second tool for a
  layer that already has one working.
- Only suggest something different when the doc actually gives a reason to:
  no tooling exists yet for that layer, or the existing tooling was flagged
  under "Known technical debt or fragile dependencies." When you do,
  name the reason from the doc, not a generic opinion about the tool (e.g.
  "rs-aut flagged the existing Selenium setup as outdated," not "Selenium is
  old").
- If this run has no `rs-aut` doc to draw from (PR-scoped only, no
  onboarding doc read), say so plainly and skip technology suggestions
  rather than guessing the stack from the repo directly.

## Session modifiers

This skill shares its session modifiers with `pr-explainer`, `rs-aut`, and
`test-suite`. Check whether a skill-level (`junior`/`mid`/`senior`) or
entry-point (`visual`/`trace`/`risk`) argument has been passed in this or an
earlier `/test-plan`, `/pr-explainer`, `/rs-aut`, or `/test-suite` invocation
this session — if any has already been set, it carries over here without
needing to be set again.
See `${CLAUDE_PLUGIN_ROOT}/reference/modifiers.md` for the full behavior of
each. If no skill level has been set yet, default to `mid` behavior and
mention briefly that a different level is available. If no entry point has
been set yet, default to plain priority order (highest-risk first) rather
than asking a clarifying question.

For this skill specifically:

- **`visual`** — group test cases by component/feature area first, then by
  priority within each group.
- **`trace`** — write the single highest-priority case first as a full
  step-by-step scenario, then list the rest more briefly.
- **`risk`** — this is already the default ordering; under this modifier,
  also state briefly why each of the top cases is highest-risk.

## Baseline rules

- Short sentences. Avoid idiom, cultural references, and unnecessarily
  complex phrasing — this is a fixed baseline for everyone, not a
  junior-only mode.
- Define acronyms on first use unless the person is clearly using them
  fluently themselves in the same message.
- Never invent or assume context you don't actually have. If the source doc
  doesn't cover something, say so plainly rather than guessing and
  presenting it as fact.
- Be concrete to this specific PR or project. Never give generic
  test-planning advice that could apply to anything.

## Working with what's available

Use only what's actually written in the source doc(s) you read. Don't pad
the plan with generic best-practice test cases that aren't grounded in
something the source doc actually flagged. If the person points you at a
specific area of the plan, focus there first.

## Always writing the result to a file

Every time you answer using this skill, also write (or update) a markdown
file with the result — automatically, without being asked. This is a
standing part of using the skill, not an optional extra someone has to
request.

- **Location** — `docs/test-plan/` at the root of the project being tested
  (not this plugin). Create the folder if it doesn't exist yet.
- **Filename** — PR-scoped runs use
  `test-plan_<identifier>_<YYYY-MM-DD>.md`, using today's actual date and the
  same `pr<number>` / sanitized-branch-name identifier convention
  `pr-explainer` uses. Project/feature-scoped runs (no PR involved) use
  `test-plan_<YYYY-MM-DD>.md`, matching `rs-aut`'s single-file-per-day
  convention. If that file already exists for the day/scope, update it in
  place rather than creating a second file.
- **Required header block** — at the top of the file, before any other
  content:
  - The creation date, written out plainly (e.g. `Created: 2026-07-28`).
  - The source doc path(s) this plan was built from.
  - A "Commands used" section listing the full command transcript so far, in
    order as actually typed — every `/test-plan` invocation verbatim,
    including whatever modifier and scope arguments were given. Render each
    as inline code. Append to this list (don't rewrite past entries) as more
    invocations happen later in the day.
- **Body** — the test plan content covered so far, organized under clear
  markdown headers (manual cases, automated case suggestions with their
  technology suggestion alongside each one, grouped by component/area when
  `visual` applies), so it scans well in an editor like VS Code rather than
  reading like a chat transcript. When a later answer covers a case already
  in the file, update it rather than duplicating it.
- After writing or updating the file, mention the path in your reply so the
  person knows it's there — but don't ask permission first, and don't make
  the write conditional on them wanting it.
