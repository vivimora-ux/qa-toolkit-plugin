---
name: test-automate
description: Turn an existing test-suite or test-plan doc into real, runnable spec files for a chosen automation framework. Tip: combine modifiers directly — e.g. /test-automate senior playwright the checkout flow. Levels: junior/mid/senior (comment density only — no visual/trace/risk). Scope: a feature area (reads docs/test-plan or docs/test-suite, whichever covers it) or blank for the most recent docs/test-suite doc.
disable-model-invocation: true
argument-hint: "[junior|mid|senior] [framework] [feature area, or blank]"
---

You help a QA or QE team member turn test cases that already exist on
paper into real, runnable automated test code — scaffolding a baseline
instead of hand-writing every spec from scratch. This is the plugin's
first skill that writes code rather than markdown analysis, and it
never invents a test case: every generated test traces back to a case
already written down in a `test-suite` or `test-plan` doc.

## Determining scope

1. If an argument names a feature or area, prefer a matching
   `docs/test-plan/` doc covering that area — it's already prioritized.
   If `test-plan` doesn't cover that area, fall back to `docs/test-suite/`.
2. If no argument is given, use the most recent file under
   `docs/test-suite/` (not just today's — the latest one that exists) —
   this is the project-wide default.
3. Don't guess a doc's location. Use the exact filename convention each
   skill uses (`test-plan_<date>.md`, `test-suite_<date>.md`) and the
   most recent relevant match.
4. If the doc this run needs doesn't exist yet, say so plainly and
   suggest running `/test-plan` or `/test-suite` first, whichever is
   missing. Stop there — don't analyze the project or diff yourself to
   fill the gap.

Treat the source doc's test cases as already-decided fact. Don't
re-derive architecture, risk, or coverage — that analysis belongs to
`rs-aut`/`pr-explainer`/`test-plan`/`test-suite`, not this skill.

## Resolving the framework

Framework choice is project-level, persistent state — not a per-session
modifier like skill level. Resolve it in this order:

1. **Explicit argument** — a recognized framework name (`playwright`,
   `webdriverio`, etc.) passed to this invocation always wins, and
   re-persists as the project's choice going forward.
2. **Previously persisted choice** — look at the most recent
   `docs/test-automate/test-automate_*.md` file's header for a
   `Framework:` line from an earlier run. Reuse it without asking again.
3. **Detected from `rs-aut`** — if neither of the above applies, default
   to whatever the most recent `rs-aut` doc's "Technology stack" section
   already found in place, the same default-to-existing-tooling rule
   `test-plan` already follows for its own automation suggestions.
4. **Ask once** — if nothing is set or detected, ask which framework to
   use, then persist the answer (write it to this run's summary doc
   header) so future runs don't ask again.

Only support a framework this plugin has conventions for in
`${CLAUDE_PLUGIN_ROOT}/reference/frameworks.md`. If the resolved
framework has no section there yet, say so and stop rather than
generating code with invented conventions.

## Identifying automatable cases

- From a `docs/test-suite/` doc: read the `Automatable` column
  (`Yes`/`No`) in each case's summary table row. Take only `Yes` cases;
  list every `No` case in the skipped report with its stated reason.
- From a `docs/test-plan/` doc: use its "Automated test case
  suggestions" section directly — cases that only appear under "Manual
  test cases" are manual-only and go in the skipped report, not treated
  as candidates.
- Never invent automatability the source doc didn't state. If a source
  doc predates this column-based format (an older `test-suite` file) and
  has no `Automatable` column, say so plainly and ask whether to treat
  all its cases as candidates or skip that doc's cases entirely — don't
  guess.

## Grouping cases by target spec file

Before generating anything, resolve where each case's code will live and
group cases that land in the same file together. This must happen
*before* any generation starts, so two agents never write the same
file.

1. For each automatable case, derive its target file from the case's
   module/feature area and the resolved framework's naming convention in
   `reference/frameworks.md` (e.g. `tests/checkout.spec.ts` for
   Playwright, `test/specs/checkout.spec.js` for WebdriverIO).
2. Before assuming that default location, check whether the project
   already has a test directory for this framework (e.g. an existing
   `tests/`, `e2e/`, `test/specs/`) and an existing spec file for that
   module. If one exists, target it (appending new cases to it) instead
   of the framework's generic default — this keeps generated code
   inside whatever structure the project already has, rather than
   creating a second, competing convention.
3. The result is one group per target file, each holding every case that
   belongs there. A file with no new cases isn't touched.

## Generating specs via Agent fan-out

For each target-file group, call the Agent tool with
`subagent_type: test-spec-writer` — one call per group, all in parallel
(single message, multiple tool calls), never two calls assigned to the same
file. Send each call:

- That file's assigned cases (their full detail block — objective,
  preconditions, test data, steps, expected result — plus priority and
  type; for a `test-plan` source, whatever detail that doc's "Automated
  test case suggestions" entry gives).
- The exact target file path, and whether it's a new file or an
  existing one being appended to.
- The resolved framework name (not the framework section itself —
  `test-spec-writer` looks that up in `reference/frameworks.md` on its own).
- The source doc's path (for its traceability comments).
- The comment-density instruction for the current skill level (see
  Session modifiers below).

`agents/test-spec-writer.md` owns the actual generation rules (framework
conventions, traceability comments, comment density, generation-time skip
handling) — this skill doesn't restate them, only supplies the per-file
inputs above.

Each agent writes its file directly and reports back: which cases it
covered, and any case it couldn't automate after all (a generation-time
skip, distinct from the pre-filtered skips above).

## Reporting

Combine both skip sources into one list — cases excluded before
generation (`Automatable: No` / manual-only) and cases skipped during
generation — each with its reason. Never merge them silently or drop
either source.

## Session modifiers

This skill's modifier scope is **asymmetric** with the rest of the
plugin: it reuses `junior`/`mid`/`senior` for comment density, the same
way skill level works everywhere else, but it does not use
`visual`/`trace`/`risk` — there's no equivalent framing concept for
writing code files. Check whether a skill-level argument was passed in
this or an earlier `/rs-aut`, `/pr-explainer`, `/test-plan`,
`/test-suite`, or `/test-automate` invocation this session; if set,
reuse it without asking again. If none has been set yet, default to
`mid`.

Pass the resolved level to every `test-spec-writer` call in this run —
what each level means for comment density is `test-spec-writer`'s own
concern (see `agents/test-spec-writer.md`), not restated here.

## Non-goals, every run

- Never pick a framework unilaterally when nothing is detected and
  nothing is persisted — ask once, per Resolving the framework above.
- Never retrofit or refactor a project's existing, substantial test
  suite — this skill scaffolds new coverage for automatable cases that
  don't have it yet; it doesn't touch or restructure existing specs
  beyond appending new cases to a matching file.
- Never run the generated tests or report pass/fail — generating code is
  this skill's job; running it is a normal dev/CI concern.
- Never invent a test case that isn't in the source `test-suite` or
  `test-plan` doc.

## Always writing the result to files

Every time you run this skill, write both outputs automatically, without
being asked — this is a standing part of using the skill.

- **Generated spec files** — written into the project's real test
  directory per the target-file resolution above. This is actual code,
  not a `docs/` analysis file, so it follows the target framework's
  normal location, not this plugin's dated-file convention.
- **Summary doc** — `docs/test-automate/test-automate_<date>.md` (create
  `docs/test-automate/` if it doesn't exist), one file per day, updated
  in place on same-day reruns rather than duplicated, matching every
  other skill's convention.
  - **Required header block**: creation date; the source doc path
    actually read; a `Framework:` line recording the resolved choice
    (this is what later runs read back per "Resolving the framework");
    a "Commands used" section listing every `/test-automate` invocation
    verbatim this session, appended to (not rewritten) as more happen.
  - **Body**: the list of generated/updated spec files with the cases
    each now covers, and the combined skipped-cases report from above,
    organized under real `##`/`###` headers.
- After writing, mention both output locations in your reply — but don't
  ask permission first, and don't make the write conditional on the
  person wanting it.
