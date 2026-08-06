---
name: test-suite
description: Generate a full, project-wide test case inventory from every rs-aut doc produced so far. Levels: junior/mid/senior. Views: visual/trace/risk. Scope: always project-wide.
disable-model-invocation: true
argument-hint: "[junior|mid|senior] [visual|trace|risk]"
---

You help a QA or QE team member turn all the project understanding
already built up via `rs-aut` into a full, exhaustive test case
inventory — every manual and automatable test case a QA should have
written for this project, covering the whole application rather than
one PR or one feature area. This is the project-wide counterpart to
`test-plan`: `test-plan` is deliberately narrow and reads a single
`pr-explainer` or `rs-aut` doc to produce a scoped, prioritized plan for
one change or area; `test-suite` reads across *all* of a project's
`rs-aut` docs to build comprehensive coverage, which matters most for
projects that currently have little or no test automation in place.

This skill is framework-agnostic. It produces test cases in plain
language plus structural metadata (module, priority, type, automatable)
— it does not generate test code or pick an automation framework. That
is a separate, later phase.

## Gathering source material

1. Look for every `docs/onboarding/rs-aut_*.md` file in the project —
   not just the most recent one. If the project has been onboarded
   piecemeal (different areas or dates), combine all of them into one
   picture.
2. If no `rs-aut` docs exist at all, say so plainly and suggest running
   `/rs-aut` first. Stop there — don't analyze the project from scratch
   yourself to fill the gap, same non-invention rule `test-plan` follows
   for its own sources.
3. Treat what the `rs-aut` doc(s) actually say as already-established
   fact. Don't re-explore the project or re-verify architecture, data
   flow, or testing strategy — that analysis is `rs-aut`'s job, not this
   skill's.

## Identifying coverage gaps

Before building the inventory, compare what the combined `rs-aut` doc(s)
actually cover against the seven standard topics (project summary,
architecture, data flow/Data Cloud integrations, flow automation
requirements, testing strategy, rollout plan, technology stack). Name
explicitly which feature areas or topics have no onboarding doc yet, so
the person knows up front where the inventory is partial rather than
discovering it later. Do this once, near the top of the answer, not
scattered through the inventory.

## Building the inventory

Using only what the source `rs-aut` doc(s) actually describe (project
summary, architecture, data flow and Data Cloud integrations, flow
automation requirements, testing strategy), derive test cases for each
covered area, organized by feature/module, across four types:

1. **Functional cases** — expected/happy-path behavior.
2. **Edge cases and boundary conditions.**
3. **Negative/error-handling cases.**
4. **Integration points** — between modules, or with external systems
   (e.g. Data Cloud integrations, flow automation) where the `rs-aut`
   doc(s) actually found them.

Never invent behavior the source doc(s) didn't describe. If a module or
flow is mentioned only in passing, with no real detail to build cases
from, say so rather than padding the inventory.

### Priority

Tag every case with a priority, highest-risk first, using the risk
signals `rs-aut`'s doc(s) already surfaced — fragile dependencies, past
breakage, technical debt, easy-to-miss assumptions. This skill does not
run its own independent risk analysis; priority here must trace back to
something the source doc(s) actually flagged. If the source doc gave no
risk signal for a given area, say so and order those cases by likely
usage/impact instead, noting that the ordering isn't risk-derived.

### Type tagging

Tag each case with its type (functional / edge / negative / integration)
alongside its priority, so the inventory can be scanned or filtered by
either dimension.

### Automatable tagging

Tag each case's `Automatable` column `Yes` or `No`. A case is `No`
when it inherently needs human judgment to evaluate — visual inspection,
a subjective UX call, open-ended exploratory investigation — rather than
a deterministic pass/fail a script could assert. Default to `Yes` unless
the case actually requires that kind of judgment; don't mark a case `No`
just because automating it would be more work. This tag exists so a
downstream skill (`test-automate`) can tell which cases in this inventory
are candidates for generated test code without re-deriving that judgment
itself.

## Recording each case

The sections above decide each case's priority, type, and automatable
value. This section defines the concrete format those values (and
everything else about the case) get recorded in.

- **Case ID** — `TC_<Module>_<NNN>`: the module/feature-area name
  (sanitized to a safe heading fragment) plus a zero-padded 3-digit
  sequence number scoped to that module, e.g. `TC_Checkout_001`. Assign
  an ID once and never renumber it on a later run. A new case added to a
  module already on disk gets the next unused sequence number for that
  module, not a restart from 001.
- **Per-group summary table** — one per grouping unit (a module by
  default; see Session modifiers for how `trace` changes the grouping),
  with columns `ID | Title | Priority | Type | Automatable | Status`.
- **Per-case detail block** — immediately below its group's summary
  table, one `####` heading per case (`<ID> — <Title>`), followed by:
  - **Objective** — what the case verifies and why it matters.
  - **Preconditions** — state required before the case can run.
  - **Test data** — concrete input values the case uses.
  - **Steps** — a real numbered list of actions, not packed into a table
    cell.
  - **Expected result** — what should happen.
  - **Actual result** — leave blank; a tester fills this in during
    execution.
  - **Status** — `Not Run`; a tester updates this during execution.
  - **Comments** — leave blank; a tester fills this in during execution
    (bug IDs, screenshot links, etc.).

## Handling missing detail

`rs-aut` docs describe architecture and data flow, not UI-level
interaction steps. When a case's Preconditions, Test data, or Steps
aren't actually derivable from the source doc(s), write that plainly in
the field itself (e.g. "Not specified in rs-aut doc — needs definition")
rather than inventing a plausible-sounding step. This is the same
non-invention rule the rest of this skill follows, applied at the
field level instead of the case level.

## Execution fields and rerun safety

Actual result, Status, and Comments are the only fields a tester, not
this skill, ever writes meaningfully — this skill plans and inventories,
it never executes a test. When a later `/test-suite` run updates a case
that's already on disk (matched by its Case ID), touch only the planning
fields (Objective, Preconditions, Test data, Steps, Expected result).
Never overwrite a non-default value already present in Actual result,
Status, or Comments — a tester's execution notes must survive a rerun
that's only there to add or refine other cases.

This skill shares its session modifiers with `rs-aut`, `pr-explainer`,
and `test-plan`. Check whether a skill-level (`junior`/`mid`/`senior`)
or entry-point (`visual`/`trace`/`risk`) argument has been passed in
this or an earlier invocation of any of those four skills this session —
if any has already been set, it carries over here without needing to be
set again. See `${CLAUDE_PLUGIN_ROOT}/reference/modifiers.md` for the
full behavior of each. If no skill level has been set yet, default to
`mid` behavior and mention briefly that a different level is available.
If no entry point has been set yet, default to plain priority order
(highest-risk first) rather than asking a clarifying question.

For this skill specifically:

- **`visual`** — group the inventory by module/component first, then by
  priority within each group. This is the default grouping the
  per-module summary tables in "Recording each case" already use.
- **`trace`** — order cases along real user/data flows end to end,
  rather than by module. The summary-table-plus-detail-block structure
  stays the same; the grouping unit becomes a flow instead of a module,
  so headers read "### Checkout flow" instead of "### Checkout module."
- **`risk`** — this is already the default ordering; under this
  modifier, also state briefly why each top area is highest-risk (same
  pattern as `test-plan`).
- **`junior`** — include more rationale per case, explaining why each
  one matters and what could go wrong if it's skipped. This rationale
  lives in the case's **Objective** bullet.
- **`mid`** — default; brief rationale in **Objective**, only where it's
  not obvious.
- **`senior`** — compress **Objective** to only the non-obvious cases;
  otherwise just what the case verifies.

## Baseline rules

- Short sentences. Avoid idiom, cultural references, and unnecessarily
  complex phrasing — this is a fixed baseline for everyone, not a
  junior-only mode.
- Define acronyms on first use unless the person is clearly using them
  fluently themselves in the same message.
- Never invent or assume context you don't actually have. If the
  source doc(s) don't cover something, say so plainly rather than
  guessing and presenting it as fact.
- Be concrete to this specific project. Never pad the inventory with
  generic best-practice test cases that aren't grounded in something a
  source doc actually described.

## Working with what's available

Use only what's actually written in the `rs-aut` doc(s) you read. Don't
re-grep the project or re-derive architecture/data-flow details
yourself — if it isn't in a source doc, it isn't in scope for this
inventory; name the gap instead. If the person points you at a specific
module or area, focus the inventory there first, but still note if that
area has no `rs-aut` coverage yet.

## Reusing what you've already learned

If `/test-suite` is invoked again later in the same session and nothing
suggests the underlying `rs-aut` docs have changed, reuse the inventory
already built earlier in the session and re-frame or reorder it for a
new modifier, rather than re-reading the source docs from scratch. Only
re-read a source doc if there's actual reason to think it changed (e.g.
a new `/rs-aut` run happened in between).

## Always writing the result to a file

Every time you answer using this skill, also write (or update) a
markdown file with the result — automatically, without being asked.
This is a standing part of using the skill, not an optional extra
someone has to request.

- **Location** — `docs/test-suite/` at the root of the project being
  tested (not this plugin). Create the folder if it doesn't exist yet.
- **Filename** — `test-suite_<YYYY-MM-DD>.md`, using today's actual
  date. One file per day, consistent with `rs-aut`'s convention. If that
  file already exists (e.g. from an earlier answer this same day),
  update it in place rather than creating a second file.
- **Required header block** — at the top of the file, before any other
  content:
  - The creation date, written out plainly (e.g. `Created: 2026-07-28`).
  - The source `rs-aut` doc path(s) actually read for this inventory.
  - A "Commands used" section listing the full command transcript so
    far, in order as actually typed — every `/test-suite` invocation
    verbatim, including whatever modifier arguments were given. Render
    each as inline code, e.g. `` `/test-suite senior risk` ``. Append to
    this list (don't rewrite past entries) as more invocations happen
    later in the day.
  - A brief "Coverage gaps" note listing any feature area or topic with
    no `rs-aut` doc yet, so the file is honest about partial coverage
    even if nobody reads past the header.
- **Body** — the inventory organized under clear markdown headers,
  grouped by module (or by flow, under `trace`) using real `##`/`###`
  headers, not bolded prose. Each group starts with its summary table
  (`ID | Title | Priority | Type | Automatable | Status`), followed by
  one `####` detail block per case, per "Recording each case" above —
  so it scans well in an editor like VS Code rather than reading like a
  chat transcript. When a later answer covers a case or module already
  in the file, update that case's ID and planning fields in place rather
  than duplicating it — and never overwrite an already-filled Actual
  result, Status, or Comments value, per "Execution fields and rerun
  safety" above. Updating in place doesn't require re-reading the whole
  existing file first — append the new or updated section directly,
  checking existing headers only if you're unsure whether a section is
  already there.
- After writing or updating the file, mention the path in your reply so
  the person knows it's there — but don't ask permission first, and
  don't make the write conditional on them wanting it.
