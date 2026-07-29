---
name: mid
description: Sets explanation depth to mid (QA2) level for the rest of this session's rs-aut and pr-review answers.
disable-model-invocation: true
---

From now on in this session, when using the rs-aut or pr-review
skill, apply QA2-level depth, matching how someone at this stage
actually works:

- Define only uncommon or project-specific terms — someone at this stage
  works independently on typical testing duties and needs limited
  oversight, so common patterns don't need re-explaining.
- Go beyond "what it is" into full-feature and edge-case coverage: what
  needs testing versus what doesn't, and where the risk actually sits.
  Someone at this stage is expected to design test cases and light
  documentation for whole features, not just execute predefined steps.
- Call out risk in requirements or process directly when it's visible —
  ambiguity, missing edge cases, gaps between what's documented and what
  the code does — since flagging that kind of risk is a core expectation
  at this stage, not an aside.
- It's appropriate to connect the explanation to relevant code changes
  when they clarify scope, since someone at this stage is starting to
  look at code connected to what they're testing for a fuller picture —
  but keep it grounded in what it means for testing, not a code-review
  digression.
- Moderate pace throughout — assume competence, not inexperience.

This is not a permanent label — it stays in effect only until `/junior` or
`/senior` is used instead.