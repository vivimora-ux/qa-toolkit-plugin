# Modifiers for `rs-aut`, `pr-explainer`, and `test-plan`

`junior`/`mid`/`senior` and `visual`/`trace`/`risk` are passed as
arguments to `/rs-aut`, `/pr-explainer`, or `/test-plan` (e.g.
`/rs-aut senior risk the checkout flow`, `/pr-explainer junior trace 142`,
`/test-plan risk 142`).
Once set, a modifier is shared across all three skills and stays in effect
for the rest of the session until a different value in the same
category is passed again. Never treat a skill-level modifier as a
permanent label — the same person may want a different level on a
different day or a different PR, depending on how familiar they already
are with the area in question. Don't remark on someone "still" using
`junior`.

## Skill level

- **`junior`** — define terms the first time they're used, and explain
  *why* something matters, not just what it is — someone at this stage
  is still building their menu of capabilities and learning the basics
  of the tools and tech in play. Assume no prior context about this
  codebase or its established processes; spell out the process/
  convention being followed, not just the technical detail, since
  someone at this stage is still learning which process applies and
  that it can differ by project. Frame explanations around concrete,
  doable next actions — what to test, what to check, what to ask about
  — rather than abstract architecture discussion. Encourage confirmation
  rather than assertion — it's appropriate to end with something like
  "if you spot something that looks off here, that's worth confirming
  with a senior QA or developer before assuming it's a bug." Reinforce
  curiosity: it's fine, and encouraged, to flag oddities or ask "why
  does it work this way?" even about things that seem settled.

- **`mid`** — define only uncommon or project-specific terms — someone
  at this stage works independently on typical testing duties and needs
  limited oversight, so common patterns don't need re-explaining. Go
  beyond "what it is" into full-feature and edge-case coverage: what
  needs testing versus what doesn't, and where the risk actually sits.
  Call out risk in requirements or process directly when it's visible —
  ambiguity, missing edge cases, gaps between what's documented and what
  the code/diff does — since flagging that kind of risk is a core
  expectation at this stage, not an aside. It's appropriate to connect
  the explanation to relevant code when it clarifies scope, since
  someone at this stage is starting to look at code connected to what
  they're testing for a fuller picture — but keep it grounded in what it
  means for testing, not a code-review digression. Moderate pace
  throughout — assume competence, not inexperience.

- **`senior`** — use precise technical vocabulary without definitions,
  and skip anything inferable from the code/diff itself — someone at
  this stage can be placed with little to no context and work
  effectively on their own. Go straight to root cause, not surface
  symptoms: what's the underlying mechanism, what's testable versus what
  resists testing, and what a post-mortem for a missed issue here would
  actually say. Frame risk in terms of quality strategy and business
  tradeoffs, not just technical fragility — e.g. where less testing
  overhead is defensible given the risk, versus where full coverage is
  non-negotiable. Surface non-obvious edge cases and testability
  concerns proactively — gaps in requirements, missing acceptance
  criteria, or design decisions that make something harder to verify —
  the way someone at this stage raises these during grooming before
  they become expensive later. It's appropriate to reference what would
  be worth documenting or teaching to others, since someone at this
  stage is expected to mentor and create walkthroughs, not just
  understand things for themselves.

## Entry point

- **`visual`** — leads with structure. In `rs-aut`, lead with what the
  pieces of the system are, how they're organized, and how they relate
  to each other — favor a spatial/structural description, or an actual
  diagram if available, over a narrative walk-through. In `pr-explainer`,
  lead with scope of impact: which files, components, or services the
  change touches, and what part of the application that represents. In
  `test-plan`, group test cases by component/feature area first, then by
  priority within each group.

- **`trace`** — leads with a step-by-step walk-through. In `rs-aut`,
  follow one concrete path through the system from start to finish, in
  the order things actually happen. In `pr-explainer`, walk through the
  meaningful logic change(s) in the diff, in the order they'd actually
  execute. In `test-plan`, write the single highest-priority test case
  first as a full step-by-step scenario, then list the rest more briefly.

- **`risk`** — leads with fragility. In `rs-aut` and `pr-explainer`: what's
  fragile, what depends on this, what's broken here before (if that's in
  loaded project knowledge), and what assumption is easiest to get wrong.
  In `test-plan`, this is already the default ordering (highest-risk
  first) — under this modifier, also state briefly why each top case is
  highest-risk.
