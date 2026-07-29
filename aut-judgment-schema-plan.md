# [Epic] Personalized schema for accelerating AUT understanding & judgment

**Type:** Epic
**Priority:** Medium-High
**Reporter:** _(unassigned — draft, no ticket number yet)_
**Team:** QA / QE
**Labels:** `ai-tooling`, `qa-enablement`, `onboarding`, `judgment`, `claude-code`

---

## Summary

Build a Claude Code plugin, installable by any team member from a shared
private marketplace repo, that helps a new QA build a fast, robust
technical understanding of a project's basics — paired with a small set of
explicit commands that let the person control how that understanding is
delivered (entry point and skill level). The goal is to externalize part of
the tacit judgment senior team members carry today, so any QA can build
real context faster, rather than relying on scripted tests or informal,
person-to-person knowledge transfer.

## Background / problem statement

The team's biggest bottleneck isn't test execution — it's judgment. Senior
team members adjudicate risk and prioritize testing based on deep,
accumulated understanding of the application under test (AUT). That
understanding is currently tacit: it lives in people's heads and is
transferred slowly, inconsistently, and informally (pairing, tribal
knowledge, time-in-seat).

This plan externalizes part of that understanding-building process into a
repeatable, installable tool — not to replace judgment, but to help people
build the contextual understanding judgment depends on, faster.

**Team context that shapes this:**
- Mixed seniority (juniors and seniors on the same team)
- Both QA (exploratory/behavioral) and QE (automation/engineering)
  perspectives are represented
- Individual learning/mindset styles differ (visual vs. sequential vs.
  risk-first thinkers)
- Some team members are non-native English speakers — content must be
  clear, literal, and low on idiom regardless of skill level

## Goals

- A working Claude Code plugin, installed locally per project via a
  shared private marketplace, so it travels with each repo rather than
  living in a separate web workspace.
- **One Skill** that covers, in general, the six things a new QA needs to
  understand about any project:
  1. Project summary
  2. Architecture overview
  3. Data flow and Data Cloud integrations
  4. Flow automation requirements
  5. Testing strategy
  6. Rollout plan
- **Six explicit commands**, layered on top of the Skill's output, giving
  the person control over how that information is delivered:
  - Entry point — `/visual`, `/trace`, `/risk`
  - Skill level — `/junior`, `/mid`, `/senior`
- Skill level is self-set and self-updatable at any time — nobody stays
  "junior" forever, and commands make recalibration a one-word action.
- Plain, unambiguous language as a baseline for every level, not a special
  mode — this serves non-native English speakers without singling anyone
  out, and costs nothing for a senior/native-English reader.
- Distributed through a private git-hosted marketplace so the whole team
  installs and updates it the same way, with real version tracking.

## Non-goals

- Not building a full public marketplace or submitting anywhere outside
  the team's private hosting.
- Not solving live PR/ticket fetching in this phase — start with the QA
  pasting in whatever artifact (code, docs, PR diff) they want explained.
- Not auto-triggering the onboarding Skill on day one — start
  explicit-only (`/aut-onboarding`) so behavior is easy to observe and
  debug; move to auto-trigger only once the description reliably fires on
  the right kind of question and false-triggering has been ruled out.
- Not solving team-wide skill-level visibility/tracking (e.g. a
  manager-facing dashboard of who's at what level).
- Not a replacement for senior mentorship/pairing — a scaffold to make
  that mentorship's effects more repeatable and available on demand.

## Proposed solution

### How this works mechanically

Claude Code has two building blocks relevant here, and as of the current
version they're technically unified: a file works as either
`.claude/commands/name.md` or `.claude/skills/name/SKILL.md`, and both
produce a `/name` shortcut. Skills add two things commands alone don't:
the ability to bundle supporting files, and frontmatter controlling
whether a person must type the command (`invocation: user`) or Claude can
apply it automatically when relevant (model-invoked). This plan uses both
patterns deliberately for different purposes:

- **One Skill** for the six-topic content — explicit-only for now, with
  auto-invocation considered later once validated.
- **Six lightweight commands** for the two dials — explicit-only by
  design, since these represent a deliberate personal choice (how someone
  wants information framed and pitched) that shouldn't be guessed at by
  the model.

A **plugin** bundles these into one installable unit via a required
manifest (`.claude-plugin/plugin.json`). A **marketplace** is a git repo
containing a `.claude-plugin/marketplace.json` that lists one or more
plugins and where to find them. Anyone on the team adds the marketplace
once and installs the plugin with two commands:

```
/plugin marketplace add vivimora-ux/<marketplace-repo-name>
/plugin install aut-onboarding@<marketplace-name>
```

When the plugin's content is updated and pushed to the repo, teammates
pull the update with:

```
/plugin marketplace update
```

This solves distribution and keeps everyone's copy of the six-topic
content in sync without manual copy-paste per repo.

**Hosting:** the private marketplace repo will live under
`https://github.com/vivimora-ux` for this demo/pilot. Exact repo name to
be decided during setup (e.g. `aut-onboarding-marketplace`).

### File structure

```
aut-onboarding-marketplace/            (git repo under vivimora-ux)
├── .claude-plugin/
│   └── marketplace.json               # lists the plugin below
└── aut-onboarding-plugin/
    ├── .claude-plugin/
    │   └── plugin.json                # name, version, description, author
    ├── skills/
    │   └── aut-onboarding/
    │       └── SKILL.md               # the six-topic content
    ├── commands/
    │   ├── visual.md
    │   ├── trace.md
    │   ├── risk.md
    │   ├── junior.md
    │   ├── mid.md
    │   └── senior.md
    └── README.md                      # install + usage instructions
```

### The Skill (`aut-onboarding`)

When invoked, walks through the six topics for whatever the QA is asking
about, at whatever depth/entry point the currently-active commands
indicate, defaulting to a balanced, `/mid`-level pass across all six
topics if no command has been set yet in the session:
1. Project summary
2. Architecture overview
3. Data flow and Data Cloud integrations
4. Flow automation requirements
5. Testing strategy
6. Rollout plan

Content must be concrete to the actual project being examined — never
generic advice that could apply to any codebase. If something isn't
available (no docs on Data Cloud integration, no written rollout plan,
etc.), the Skill should say so plainly rather than guessing.

### The six commands

- `/visual` — leads with structure and shape: what the pieces are, how
  they're organized, how they relate.
- `/trace` — leads with a step-by-step walk-through of one concrete
  execution path.
- `/risk` — leads with what's fragile: what depends on this, what's
  broken here before, what assumption is easiest to get wrong.
- `/junior` — defines terms on first use, explains why something matters,
  assumes no prior context, may check understanding before moving on.
- `/mid` — defines only uncommon or project-specific terms, moderate
  pacing.
- `/senior` — precise vocabulary, no definitions, compressed to
  non-obvious content and edge cases.

Tags are never treated as a permanent label — a person may use a
different skill-level command on a different day depending on how
familiar they are with the specific area in question, and the plugin
should not remark on someone "still" using `/junior`.

### Baseline behavior, regardless of command used

- Short sentences, minimal idiom, no unnecessary cultural references —
  fixed baseline for everyone, not a junior-only mode.
- Define acronyms on first use unless the person is clearly using them
  fluently themselves.
- Never invent or assume context not actually available — say so
  explicitly when something is unknown.

## Acceptance criteria

- [ ] `aut-onboarding` Skill drafted, covering all six topics with
      concrete, project-specific content
- [ ] Six command files drafted (`visual`, `trace`, `risk`, `junior`,
      `mid`, `senior`), each a short, single-purpose modifier
- [ ] `plugin.json` and `marketplace.json` written and valid
- [ ] Reviewed by at least 2 senior team members for accuracy of what a
      new QA actually needs to know first
- [ ] Commands genuinely change the Skill's output (framing for
      entry-point commands, depth/vocabulary for skill-level commands),
      not just cosmetic differences
- [ ] No-command invocation (`/aut-onboarding` alone) still produces a
      usable, balanced response
- [ ] Marketplace repo hosted under `github.com/vivimora-ux`; at least one
      teammate successfully installs via `/plugin marketplace add` +
      `/plugin install` from documentation alone
- [ ] Piloted by at least one new/junior QA and one senior QA on the same
      real project
- [ ] Plain-language baseline holds under review regardless of which
      skill-level command was used
- [ ] A content update is pushed and successfully pulled by a teammate via
      `/plugin marketplace update`, confirming the update path works

## Open questions / risks

- **Auto-trigger criteria (deferred, not decided):** once explicit
  invocation is validated, what description makes the Skill fire
  reliably on "help me understand this project" style questions without
  false-triggering on unrelated requests?
- **Six topics require real source material to be accurate.** Data flow /
  Data Cloud integrations and flow automation requirements in particular
  will need actual project documentation or code access to answer
  correctly.
- **Content maintenance ownership.** Someone needs to own updating the
  Skill's content as projects evolve, and pushing version bumps to the
  marketplace repo — otherwise installed copies go stale across the team.
- **Private repo access.** Since the marketplace lives under a personal
  private GitHub account (`vivimora-ux`) for this demo, team-wide rollout
  will eventually need either shared access to that account/repo or a
  move to a company-owned org repo — worth deciding before full rollout,
  even though it's fine for the pilot.

## Suggested subtasks

1. Draft `aut-onboarding/SKILL.md` covering the six topics
2. Draft the six command files (three entry-point, three skill-level)
3. Write `plugin.json` and `marketplace.json`
4. Create and push the marketplace repo under `github.com/vivimora-ux`
5. Internal review with 2+ seniors for content accuracy
6. Install via `/plugin marketplace add` + `/plugin install` on one real
   project; pilot with 1 new QA + 1 senior QA on that same project
7. Revise Skill/commands based on pilot feedback; push an updated version
   and confirm `/plugin marketplace update` works for a teammate
8. Write setup/install doc for the rest of the team
9. Decide on long-term repo hosting (personal vs. company org) before
   full team rollout

## Definition of done

The plugin — Skill, six commands, and manifests — is drafted, reviewed by
senior team members, hosted on a private marketplace repo under
`github.com/vivimora-ux`, successfully installed and piloted by both a
new/junior QA and a senior QA on a real project, revised based on that
feedback, and documented well enough that another teammate can install and
update it without a walkthrough.
