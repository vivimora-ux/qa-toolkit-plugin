# Testing guide: `aut-onboarding` and `pr-review`

For QA testers verifying the `/aut-onboarding` and `/pr-review` skills,
and the six shared modifier commands, on a project of your choice.

## Prerequisites

- Pick a target project (not this plugin repo) to test in. It should
  have real git history and at least one real PR, or an unmerged local
  branch with changes — the `pr-review` tests need something to diff
  against.
- In that project, install the plugin from inside Claude Code:

```
/plugin marketplace add vivimora-ux/aut-onboarding-marketplace
/plugin install aut-onboarding@aut-onboarding-marketplace
```

## What you're testing

- **2 skills**: `/aut-onboarding [topic]`, `/pr-review [PR#|URL|blank]`
- **6 modifier commands**, shared across both skills, in effect for the
  rest of the session once set:
  - Skill level: `/junior`, `/mid`, `/senior`
  - Entry point: `/visual`, `/trace`, `/risk`

Replace `<PR#>`, `<PR-URL>`, and `<topic>` below with real values from
your target project.

## Test 1 — Defaults (no modifiers set)

```
/aut-onboarding help me understand this project
```

Expect: mid-level depth, a balanced pass (structure → trace → risk),
and a brief mention that other levels exist. Check that
`docs/onboarding/aut-onboarding_<today's date>.md` is created in the
target project, with a header block (`Created:`, `Commands used:`) and
real `##` headers.

## Test 2 — Each modifier individually, on `/aut-onboarding`

Run these one at a time, in a fresh session each, to confirm each
behaves distinctly:

```
/junior
/aut-onboarding the main data flow
```

```
/senior
/aut-onboarding the main data flow
```

```
/visual
/aut-onboarding architecture
```

```
/trace
/aut-onboarding the main data flow
```

```
/risk
/aut-onboarding testing strategy
```

Expect visibly different tone/framing per command, not just a label
change:
- `/junior` defines terms and ends with a confirmation-style check-in.
- `/senior` skips definitions and goes straight to edge cases.
- `/visual` leads with architecture.
- `/trace` leads with a step-by-step walk-through.
- `/risk` leads with fragility/coverage gaps.

## Test 3 — Combining modifiers, one session

```
/senior /risk
/aut-onboarding <topic>
```

Expect senior vocabulary and risk-first framing combined in the same
answer.

## Test 4 — Modifiers persist and carry over to `/pr-review`

In the *same* session as Test 3 (don't reset), without resetting the
modifiers, switch skills:

```
/pr-review
```

Expect it still applies `/senior /risk` from earlier — risk-led, no
re-explaining terms — confirming that modifiers are shared across both
skills.

## Test 5 — `/pr-review` input sources

Run each in its own session (reset modifiers with `/mid` first if you
want a clean baseline):

- `/pr-review <PR#>` — should use `gh pr view` / `gh pr diff`.
- `/pr-review <PR-URL>` — same, via URL.
- `/pr-review` with no argument, on a branch with local changes —
  should diff against the default branch.
- `/pr-review` with no argument, on a clean branch with nothing to
  compare — should say plainly there's nothing to review, not invent
  content.
- (Optional) simulate `gh` not being authenticated (`gh auth logout`
  temporarily, only in a throwaway/safe environment) — should say so
  plainly and fall back to the local diff instead of failing silently.

## Test 6 — The seven-topic coverage in `/pr-review`

On one real PR, check the answer actually hits: change summary, scope
of impact, code walk-through, test coverage (explicitly flagging logic
changes with no accompanying test changes), risk assessment, suggested
test plan (priority-ordered), and non-functional considerations
(skipped briefly if not applicable, not omitted silently).

## Test 7 — File output correctness

After a few runs on the same day:

- Confirm `docs/onboarding/aut-onboarding_<today's date>.md` and
  `docs/pr-reviews/pr-review_pr<PR#>_<today's date>.md` (or
  `pr-review_<branch>_<today's date>.md`) exist in the *target*
  project, not this plugin repo.
- Run a second `/aut-onboarding` or `/pr-review` later the same day on
  the same topic/PR — confirm it **updates** the existing file
  (appends to "Commands used", updates matching sections) rather than
  creating a duplicate file.
- Confirm the "Commands used" list accumulates every modifier and
  invocation typed so far, in order, as inline code.

## Quick pass/fail checklist

- [ ] Plugin installs cleanly in a separate project
- [ ] Each of the 6 commands produces visibly distinct behavior
- [ ] Modifiers persist across the session and across both skills
- [ ] `/pr-review` handles: PR number, PR URL, blank/local diff,
      no-`gh`, nothing-to-review
- [ ] Files are written to the target project's `docs/onboarding/` and
      `docs/pr-reviews/`, not this repo
- [ ] Same-day reruns update files in place instead of duplicating
