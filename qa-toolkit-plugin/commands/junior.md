---
name: junior
description: Sets explanation depth to junior (QA1) level for the rest of this session's rs-aut and pr-review answers.
disable-model-invocation: true
---

From now on in this session, when using the rs-aut or pr-review
skill, apply QA1-level depth, matching how someone at this stage
actually works:

- Define terms the first time they're used, and explain *why* something
  matters, not just what it is — someone at this stage is still building
  their menu of capabilities and learning the basics of the tools and
  tech in play.
- Assume no prior context about this codebase or its established
  processes. Spell out the process/convention being followed, not just
  the technical detail, since someone at this stage is still learning
  which process applies and that it can differ by project.
- Frame explanations around concrete, doable next actions — what to test,
  what to check, what to ask about — rather than abstract architecture
  discussion, since the priority at this stage is completing assigned
  tasks accurately and asking for help when something is unclear.
- Encourage confirmation rather than assertion. It's appropriate to end
  with something like "if you spot something that looks off here, that's
  worth confirming with a senior QA or developer before assuming it's a
  bug" — mirroring how someone at this stage is expected to verify
  findings rather than declare them.
- Reinforce curiosity: it's fine, and encouraged, to flag oddities or ask
  "why does it work this way?" even about things that seem settled.

This is not a permanent label — it stays in effect only until `/mid` or
`/senior` is used instead.