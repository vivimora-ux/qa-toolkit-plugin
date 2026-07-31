# [Story] PR / recent-changes review skill (Phase 2)

**Type:** Story (part of the "Personalized schema for accelerating AUT
understanding & judgment" epic)
**Priority:** Medium-High
**Reporter:** _(unassigned — draft, no ticket number yet)_
**Team:** QA / QE
**Labels:** `ai-tooling`, `qa-enablement`, `claude-code`, `pr-review`

---

## Summary

Add a second Skill to the existing `aut-onboarding` plugin that lets a QA
point Claude at a set of recent changes — a PR, a branch diff, or
uncommitted work — and get a judgment-focused analysis of *only what
changed*, instead of the whole-project sweep the onboarding skill does.
Reuses the same skill-level commands (`/junior`, `/mid`, `/senior`) from
Phase 1; does not reuse the onboarding skill's seven-topic structure,
since this answers a narrower, different question.

## Background

Phase 1 (`aut-onboarding`) helps someone understand a whole project.
That's the wrong shape of answer when the actual need is "a PR just
landed — what does it do, and what should I go test?" This phase revives
and formalizes that original PR-judgment idea, now built to actually run
inside Claude Code, where it has direct local `git` access — a real
advantage over the earlier paste-the-diff-into-a-chat approach.

## Decisions made so far

- **Scope of "recent changes" is not fixed** — the person specifies it
  each time (e.g. "diff against main," "PR #42," "just my staged
  changes"). The skill needs to interpret whatever the person says and
  run the right `git` command, rather than assuming one default.
- **PR metadata matters, not just the code diff.** Title, description, and
  any linked ticket carry intent that a diff alone doesn't show (e.g. *why*
  a change was made). This will be pulled in via a GitHub connector
  (MCP), not just local git.

## Goals

- New Skill: `pr-review`, added to the existing plugin alongside
  `aut-onboarding`.
- Accepts a scope described in plain language (a branch name, a PR
  number, "staged changes," "since yesterday," etc.) and resolves it to
  the right underlying `git diff` command itself, asking for
  clarification only if it genuinely can't tell what's meant.
- Pulls PR title, description, and linked ticket (when a PR number or
  link is given) via a GitHub MCP connector, and combines that with the
  code diff before analyzing.
- Covers, for whatever the scope resolves to:
  1. What actually changed, in plain terms (behavior change vs. refactor
     vs. bug fix vs. mix)
  2. Blast radius — what else in the repo touches the changed code
  3. Riskiest assumptions in the change, and what breaks if they're wrong
  4. Connection to past fragility in this area, if that's discoverable
     from repo history or docs
  5. Concrete, specific things worth manually testing
- Reuses `/junior`, `/mid`, `/senior` from Phase 1 unchanged — same depth
  behavior, applied to this narrower content instead of the seven topics.
- Entry-point commands (`/visual`, `/trace`, `/risk`) continue to work,
  reinterpreted for a diff: `/risk` leads with items 3–4 above, `/trace`
  walks the diff in order, `/visual` leads with what changed structurally.

## Non-goals

- Not fetching PR review comments, CI/CD status, or approval state —
  scope is the diff plus the PR's own title/description/linked ticket.
- Not auto-triggering on PR events (e.g. running automatically when a PR
  opens) — this stays explicitly invoked, same as `aut-onboarding`.
- Not resolving merge conflicts or suggesting code changes — this is an
  understanding/judgment tool, not a code-review or fix-it tool.
- Not deciding pass/fail on a PR — output informs a human's judgment call,
  it doesn't replace it.

## Proposed solution

### File structure additions

```
aut-onboarding-plugin/
├── .claude-plugin/
│   └── plugin.json            # version bump, updated description
├── .mcp.json                  # NEW — GitHub MCP connector config
├── skills/
│   ├── aut-onboarding/
│   │   └── SKILL.md           # unchanged
│   └── pr-review/
│       └── SKILL.md           # NEW
└── commands/                  # unchanged — visual/trace/risk/junior/mid/senior
                                #  reused as-is by both skills
```

### The Skill (`pr-review`)

When invoked, first resolves scope:
- If the person names a scope ("diff against main," "PR #42," "staged
  changes," a branch name), translate that into the right git command
  (`git diff main...HEAD`, `git diff --staged`, etc.) or, for a PR
  number/link, use the GitHub connector to fetch the PR's diff and
  metadata directly.
- If no scope is given at all, ask one direct question rather than
  guessing — unlike the onboarding skill, guessing wrong here could mean
  analyzing the wrong change entirely.

Then works through the five items in Goals, applying whichever
skill-level and entry-point commands are currently active (same
mechanism as `aut-onboarding` — modifiers persist for the session).

Same baseline rules as Phase 1 apply: plain language, no invented
context, concrete to the actual diff rather than generic advice, and say
plainly when something (like historical fragility) isn't discoverable
rather than guessing.

### GitHub connector

Add an `.mcp.json` to the plugin bundling GitHub's MCP server, so PR
metadata and diffs can be fetched directly rather than requiring the
person to have already checked out the branch locally. Exact server
endpoint and auth method (personal access token vs. OAuth, and whether
org approval is needed for a private repo) need to be confirmed during
implementation — this is flagged as an open risk below.

## Acceptance criteria

- [ ] `pr-review/SKILL.md` drafted, covering all five content areas
- [ ] Scope resolution correctly handles at least: a named base branch, a
      PR number/link, and "staged/uncommitted changes"
- [ ] GitHub connector successfully pulls PR title, description, and
      linked ticket for at least one real PR in the pilot repo
- [ ] `/junior`, `/mid`, `/senior` produce meaningfully different depth on
      PR content, same as they do for onboarding content
- [ ] `/visual`, `/trace`, `/risk` produce genuinely different framing of
      the same diff, not just reordered text
- [ ] No-scope invocation asks one clear clarifying question instead of
      guessing
- [ ] Piloted against at least 3 real PRs of varying size/risk, reviewed
      by a senior for accuracy of what it flagged vs. missed
- [ ] Plugin version bumped and pushed; a teammate confirms
      `/plugin marketplace update` picks up the new skill

## Open questions / risks

- **GitHub connector auth and access.** Needs a decision on personal
  access token vs. OAuth-based connector, and whether the private
  pilot repo requires org-level approval to connect — this blocks the
  metadata-fetching part specifically, not the local-diff part.
- **Scope-language ambiguity.** Natural-language scope resolution ("since
  yesterday," "my last few commits") is inherently fuzzier than a fixed
  default — worth watching during pilot whether it misinterprets scope
  often enough to need a stricter fallback (e.g. always confirm the
  resolved git command before analyzing, at least at first).
- **False confidence on blast radius/history.** Same risk noted in Phase
  1 — if the repo doesn't have rich history/docs, items 2 and 4 will be
  thin. The skill should say so rather than compensate by guessing.
- **Overlap with existing code-review tooling.** If the team already uses
  something like a GitHub Copilot PR summary or CodeRabbit, worth
  clarifying this tool's role is judgment/understanding-building for the
  QA specifically, not a replacement for automated review comments.

## Suggested subtasks

1. Draft `pr-review/SKILL.md`
2. Configure and test the GitHub MCP connector (`.mcp.json`), resolve
   auth/access questions
3. Test scope resolution against a handful of real branches/PRs locally
4. Internal review with 2+ seniors on 3+ real PRs — compare tool output
   to what they'd flag themselves
5. Revise based on review feedback
6. Bump plugin version, update README, push to marketplace repo
7. Confirm update path works for a teammate via `/plugin marketplace
   update`

## Definition of done

`pr-review` is drafted, its scope-resolution and GitHub connector both
work against real PRs in the pilot repo, it's been reviewed by senior
team members against at least 3 real PRs, revised based on that
feedback, and shipped as an update to the existing marketplace plugin.
