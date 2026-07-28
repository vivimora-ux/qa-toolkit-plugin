---
name: senior
description: Sets explanation depth to senior (SQA1/SQA2) level for the rest of this session's AUT-onboarding answers.
disable-model-invocation: true
---

From now on in this session, when using the aut-onboarding skill, apply
senior-level depth, matching how someone at this stage actually works:

- Use precise technical vocabulary without definitions, and skip anything
  inferable from the code itself — someone at this stage can be placed on
  a project with little to no context and work effectively on their own.
- Go straight to root cause, not surface symptoms: what's the underlying
  mechanism, what's testable versus what resists testing, and what would
  a post-mortem for a missed issue here actually say.
- Frame risk in terms of quality strategy and business tradeoffs, not just
  technical fragility — e.g. where less testing overhead is defensible
  given the risk, versus where full coverage is non-negotiable. Someone
  at this stage is expected to act as a quality strategist, not just
  report pass/fail.
- Surface non-obvious edge cases and testability concerns proactively —
  gaps in requirements, missing acceptance criteria, or design decisions
  that make something harder to verify — the way someone at this stage
  raises these during grooming before they become expensive later.
- It's appropriate to reference what would be worth documenting or
  teaching to others, since someone at this stage is expected to mentor
  and create walkthroughs, not just understand things for themselves.

This is not a permanent label — it stays in effect only until `/junior` or
`/mid` is used instead.