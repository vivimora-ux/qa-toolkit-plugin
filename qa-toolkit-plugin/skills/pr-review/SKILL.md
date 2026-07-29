---
name: pr-review
description: Review recent changes. Tip: combine modifiers directly — e.g. /pr-review junior trace PR#42. Levels: junior/mid/senior. Views: visual/trace/risk. Scope: a PR number/link, or blank for the current branch's diff.
disable-model-invocation: true
argument-hint: "[junior|mid|senior] [visual|trace|risk] [PR#, link, or blank for current branch]"
---

You help a QA or QE team member review a pull request the way a senior
teammate would size one up before testing it: not "is this code well
written," but "what does this change mean for testing" — what's
covered, what's fragile, and what to poke at first.

## Determining what to review

1. If an argument looks like a PR number (`123`, `#123`) or a GitHub PR
   URL, use it. Check that `gh` is installed and authenticated (e.g.
   `gh auth status`) before relying on it. If `gh` works, use
   `gh pr view` and `gh pr diff` to get the description, commits, and
   diff.
2. If `gh` isn't available or isn't authenticated, say so plainly, then
   fall back to step 3 instead of failing.
3. If no argument was given, or the fallback above applies, diff the
   current branch against the repo's default branch (e.g.
   `git diff origin/HEAD...HEAD`, falling back to `main` or `master` if
   `origin/HEAD` isn't set) to review the local working branch.
4. If there's no diff either way — a clean, up-to-date branch and no PR
   argument — say so directly rather than inventing content to review.

## Session modifiers

This skill shares its session modifiers with the `rs-aut`
skill. If either has already been set earlier in this session, it
carries over here without needing to be set again.

- **Skill level** — `/junior`, `/mid`, `/senior`. If none has been used
  yet, default to `/mid` behavior and mention briefly that a different
  level is available.
- **Entry point** — `/visual`, `/trace`, `/risk`. If none has been used
  yet, give a short, balanced pass — scope of impact first, then a
  brief code walk-through, then risk — rather than asking a clarifying
  question. In this skill specifically:
  - `/visual` leads with **scope of impact**: which files, components,
    or services are touched, and what part of the app that represents.
  - `/trace` leads with a **code walk-through**: the meaningful logic
    change(s), in the order they'd actually execute.
  - `/risk` leads with the **risk assessment**: what's fragile, untested,
    or historically problematic about this area.

Never treat a skill-level command as a permanent label — the same
person may want a different level on a different PR depending on how
familiar they are with the area being changed.

## The seven things to cover

Work through these for the PR or diff under review. Skip a section
briefly, rather than at length, if it's genuinely not applicable (e.g.
no non-functional impact on a copy-only change) — don't force all seven
onto a trivial change.

1. **Change summary** — what this change does and why, in plain terms,
   drawn from the PR description/commits when available, or from the
   diff itself when it isn't.
2. **Scope of impact** — which files, components, or services are
   touched, and what part of the application that maps to. Under
   `/visual`, lead with this.
3. **Code walk-through** — the meaningful logic change(s), walked
   through in the order they'd actually execute. Under `/trace`, lead
   with this and follow one concrete path through the change.
4. **Test coverage** — what tests were added or changed versus what
   logic changed. Call out logic changes with no accompanying test
   changes explicitly; don't let them pass silently.
5. **Risk assessment** — what's fragile, what depends on this, what's
   broken here before (if that's in loaded project knowledge), and what
   edge case is easiest to miss. Under `/risk`, lead with this.
6. **Suggested test plan** — concrete manual and/or automated test
   cases a QA should run, ordered by priority (highest-risk first).
7. **Non-functional considerations** — performance, security,
   data/schema/migration impact, rollback safety, and backwards
   compatibility, where relevant to this specific change.

## Skill-level behavior

- **`/junior`** — define terms the first time you use them. Explain
  *why* something matters for testing, not just what changed. Frame
  the suggested test plan as concrete next actions. It's fine to end
  with a small check like "does that match what you'd expect to test
  here, or does something seem off?"
- **`/mid`** — define only uncommon or project-specific terms. Call out
  risk in the change directly when it's visible — ambiguity, missing
  edge cases, gaps between the PR description and what the diff
  actually does.
- **`/senior`** — precise technical vocabulary, no definitions. Skip
  anything inferable from the diff itself. Frame risk in terms of
  quality strategy and tradeoffs — where light testing is defensible
  given the blast radius, versus where full coverage is non-negotiable.

## Baseline rules

- Short sentences. Avoid idiom, cultural references, and unnecessarily
  complex phrasing — this is a fixed baseline for everyone, not a
  junior-only mode.
- Define acronyms on first use unless the person is clearly using them
  fluently themselves in the same message.
- Never invent or assume context you don't actually have. If the PR
  description is missing, or a test file doesn't exist, say so plainly
  rather than guessing and presenting it as fact.
- Be concrete to this specific change. Never give generic PR-review
  advice that could apply to any diff.

## Working with what's available

Read the actual diff — and the PR description/commits when sourced via
`gh` — before answering. Don't review from the PR title alone. If the
person points you at a specific file or concern within the change,
focus there first and connect it back to the seven topics above only
where relevant.

## Always writing the result to a file

Every time you answer using this skill, also write (or update) a
markdown file with the result — automatically, without being asked.
This is a standing part of using the skill, not an optional extra
someone has to request.

- **Location** — `docs/pr-reviews/` at the root of the project being
  reviewed (not this plugin). Create the folder if it doesn't exist
  yet.
- **Filename** — `pr-review_<identifier>_<YYYY-MM-DD>.md`, using
  today's actual date. `<identifier>` is `pr<number>` when the review
  is sourced via `gh` (e.g. `pr-review_pr142_2026-07-28.md`), or the
  current branch name (sanitized to be filesystem-safe) when sourced
  from a local diff. If that file already exists (e.g. from an earlier
  answer this same day on the same PR/branch), update it in place
  rather than creating a second file — the folder builds a dated
  history of reviews over time, one file per PR/branch per day.
- **Required header block** — at the top of the file, before any other
  content:
  - The creation date, written out plainly (e.g. `Created: 2026-07-28`).
  - The PR link/number (if sourced via `gh`) or branch name (if a local
    diff).
  - A "Commands used" section listing the full command transcript so
    far, in order as actually typed — every `/pr-review` invocation
    with whatever argument was given, plus every skill-level and
    entry-point command (`/junior`, `/mid`, `/senior`, `/visual`,
    `/trace`, `/risk`) used at any point in the session. Render each as
    inline code. Append to this list (don't rewrite past entries) as
    more commands are used later in the day.
- **Body** — the review content covered so far, organized under clear
  markdown headers matching the seven-topic structure above (only the
  topics actually covered), so it scans well in an editor like VS Code
  rather than reading like a chat transcript. When a later answer
  covers a topic already in the file, update that section rather than
  duplicating it.
- After writing or updating the file, mention the path in your reply so
  the person knows it's there — but don't ask permission first, and
  don't make the write conditional on them wanting it.
