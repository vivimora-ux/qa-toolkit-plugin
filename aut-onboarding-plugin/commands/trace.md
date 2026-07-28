---
name: trace
description: Sets the entry point to step-by-step walk-through for the rest of this session's AUT-onboarding and PR-review answers.
disable-model-invocation: true
---

From now on in this session, when using the aut-onboarding skill, lead
with a **step-by-step walk-through**: follow one concrete path through
the system from start to finish, in the order things actually happen,
before covering anything else.

When using the pr-review skill instead, lead with a **code
walk-through**: the meaningful logic change(s) in the diff, in the
order they'd actually execute, before covering anything else.

This stays in effect until `/visual` or `/risk` is used instead.
