---
name: spec-driven-development
description: Guidance for running spec-driven development (SDD) on a software project using Claude, Claude Code, and (if the project has a UI) Claude Design together — writing a constitution and specs before code exists, translating design references into implementation, and running a disciplined build-and-review loop. Use this whenever starting a new app, website, or software project from scratch with Claude Code as the implementer; when the user mentions "spec-driven development," "SDD," writing a CLAUDE.md/constitution, or wants a structured spec → plan → tasks → implement workflow; or when picking up an existing SDD project and needing to know how the pieces fit together. Also use when the user is deciding how to divide work between a chat-based planning conversation, Claude Design, and Claude Code, or asking how to review AI-written code without becoming a bottleneck.
---

# Spec-Driven Development with Claude

A methodology for building real software with Claude as the planning
partner and Claude Code as the implementer, refined across a full
project build (an iOS app, from blank repo to a working, tested,
CloudKit-synced v1). Nothing here is specific to that app — it's the
process, not the product.

## The core idea

The chat conversation where a spec gets written and the Claude Code
session where it gets built don't share memory. **The repo is the only
real interface between them.** Every decision that needs to survive past
one conversation has to end up in a file, or it's gone the moment either
session ends. This one fact drives almost everything else in this skill:
which documents exist, why `plan.md` matters more over time than `spec.md`
does, why context can be cleared aggressively without losing anything
real, and why a project that documents its reasoning well is *easier* to
resume cold than one that doesn't.

## The three-tool division of labor

- **Claude (chat)** is the planning partner at a project's start —
  narrowing scope, resolving ambiguity, arguing about a design
  decision, asking the question that saves a rewrite later. It hosts
  the idea conversation, the constitution, and the first `spec.md`,
  none of which has a codebase to look at yet, and early in a project
  it also authors `plan.md` and `tasks.md`. Once the project has
  shipped code, every later spec conversation moves into Claude Code
  too (see "Spec conversations" under "Model tiering"), and plan and
  task authorship moves to the `sdd-planner` (see "Who authors plan.md
  and tasks.md" below).
- **Claude Design** (if the project has a UI) produces visual references
  — screens, a token system, a written brief — not literal source code
  for a native app. Its output is HTML/CSS underneath. For a web app that
  might be directly usable; for anything else (iOS, desktop, etc.) treat
  it as what a designer's mockups would be for a native team: the target
  to translate toward, not something to import.
- **Claude Code** implements, tests, and verifies against the documents
  as ground truth. It should be reading `CLAUDE.md` at the start of every
  session and treating `spec.md`/`plan.md`/`tasks.md` as the actual
  source of truth for what's being built — not the chat history. Once
  shipped code exists, it also drafts `plan.md` and `tasks.md` — through
  the `sdd-planner` subagent, one dispatch per spec — since at that
  point the ground truth a plan extends lives where only it can see.
  During implementation it orchestrates rather than
  types: each routine task is dispatched to the `sdd-implementer`
  subagent one tier down, and the main session triages, verifies, and
  commits (see "Model tiering" below).

## Involvement level: decide it once, at the start

The person's role in a project is a choice made in the constitution
conversation and written into `CLAUDE.md`, not something re-derived at
every pause. Two levels:

- **Product owner** (the default). The person owns `spec.md` — that's
  where most of their involvement lives — attests to behavior by using
  the app at phase pauses, and decides the two escalation triggers.
  They never approve technical work: `plan.md` and `tasks.md` are
  drafted by the sdd-planner and signed off by the skeptical-reviewer,
  and each phase (and any task the planner marked for its own review)
  is reviewed by the skeptical-reviewer rather than by the person. What
  reaches them is a spec-conformance summary, not an architecture
  review.
- **Technical lead**. The person also reads and approves `plan.md` and
  `tasks.md`, and implementation pauses for their review after every
  task the planner marked `review: per-task` — the original shape of
  this skill, with the per-task gate now the exception rather than the
  rule for foundational phases.

Everything below that mentions a review, an approval, or a pause is
written for the product-owner level unless it says otherwise; the
technical-lead variant is the same flow with the person added back at
those gates. The default is product owner because, across the projects
this skill has run on, a person who set out not to touch code turned
out not to want to approve architecture either — the per-task pauses
became a report glanced at and a button pressed, which is worse than no
gate: it looks like review without being one.

## The document set

- **`CLAUDE.md`** — the constitution. Lives at the repo root, gets read
  automatically at the start of every Claude Code session. Platform
  choices, architecture rules, testing requirements, git conventions —
  and *why*, not just the rule. This file accumulates the project's
  hard-won engineering lessons as they're discovered (see "Principles
  worth generalizing" below) and becomes more valuable over time, not
  less.
- **`specs/<NNN>-<slug>/spec.md`** — what and why. User-facing behavior,
  acceptance criteria, explicit non-goals. No implementation detail —
  that discipline matters, because a spec full of implementation detail
  stops being a place to argue about *behavior* and starts constraining
  a plan that hasn't been written yet.
- **`specs/<NNN>-<slug>/plan.md`** — technical design. This is the
  document that ends up mattering most, months in: not just what got
  built, but *why*, including the decisions that got reversed and the
  bugs that changed the design. Treat it as a living record, not a
  one-time handoff artifact — update it whenever a real decision gets
  made or corrected, not just at the start of a spec.
- **`specs/<NNN>-<slug>/tasks.md`** — ordered, small, independently
  verifiable execution steps. See "The multi-writer file problem" below
  — this file needs different handling than the others the moment
  implementation starts.
- **`DECISIONS.md`** (repo root) — business, product, and process context
  that doesn't fit the structured docs above: naming rationale, legal or
  business decisions, tooling choices. The dumping ground that keeps
  `plan.md` from accumulating things that aren't actually technical
  design.
- **`ROADMAP.md`** (repo root) — the backlog of future specs. Deliberately
  *not* ordered or scheduled — priorities should be set once there's a
  working app to actually use, not guessed at from a backlog.
- **`design/brief.md`** (if the project has a UI) — visual and
  interaction direction for whatever design process is being used,
  referencing `spec.md` for functional detail rather than restating it.
  See "Design exploration" below for what actually needs to be in it.

## Design exploration, when the project has a UI

A written design brief precedes any actual screen design — `assets/
design-brief-template.md` is the starting shape, deliberately empty of
any specific project's actual answers (see the note at the top of that
file for why copying a previous project's palette or signature element
forward defeats the point). Four principles are worth stating here
directly, since on the one project this has been tried on, they were the
difference between a distinctive visual identity and a generic one:

1. **Draw from the audience's own existing visual vocabulary, not a
   generic aesthetic.** Before reaching for a trendy look, ask whether
   this specific audience already sees, uses, or handles something in
   their own world that could inform the visual language authentically.
   A gear-hobbyist audience, for instance, already has a real vocabulary
   — aperture rings, VU meters, brass hardware — that's authentic to
   them rather than invented from a mood board. Ask the equivalent
   question for whoever the actual audience is; not every audience has
   an obvious answer, but it's worth genuinely checking first.

2. **Name and rule out the current AI-generated design defaults,
   explicitly, by name.** The single highest-leverage line in a design
   brief — and one with a short shelf life, since what's over-used in
   AI-generated design shifts over time. Look at what's actually common
   right now when writing a new brief, rather than reusing a previous
   project's list; the durable part is the habit, not any specific set
   of clichés.

3. **Look for one signature element — recurring, functional,
   distinctive — and don't force one where none exists.** The highest-
   value single decision in a visual identity, when a real candidate
   exists: something that shows up in multiple places, does real work
   rather than existing purely as decoration, and would be recognizable
   as belonging to this product from a screenshot of the element alone.
   Forcing one where the product has no natural candidate produces
   arbitrary decoration instead.

4. **If aiming for "flat but characterful" rather than purely generic-
   flat, state the skeuomorphism boundary explicitly.** Physical or
   tactile objects can supply shape and metaphor language without
   crossing into photorealistic rendering — genuinely hard to hit
   consistently unless the brief draws that line on purpose.

## The first-spec exception

A brand-new project's first spec is usually going to be bigger and more
coupled than every spec after it — the data model, the core screens, and
the core flows are all interdependent, so splitting them into separate
specs before any of them work yet just adds coordination overhead with
zero payoff. This is a deliberate, one-time exception, not a failure to
scope well. The moment that first spec ships, the condition that
justified it stops holding — there's now an established codebase, and
every spec after it should return to the normal rule: one feature, one
spec, sized to be reviewable on its own.

## Git conventions

- One branch per **spec**, never per task or phase.
- Open the PR as a **draft** immediately after pushing the branch — it
  gives a running diff to review commit-by-commit, separate from
  whatever Claude Code's own summaries say. This is genuinely useful,
  not just process for its own sake.
- Only mark it ready and merge once *every* task in that spec's
  `tasks.md` is done and verified — not when it "looks done." Merging
  partway through, even with good intentions, defeats the point of
  scoping a spec as one coherent unit.
- Marking ready also has a close-out step: update `ROADMAP.md` (drop
  or annotate what the spec shipped, add follow-ups it surfaced) and
  the repo README (if user-facing behavior or setup changed), then run
  the pre-merge review sweep so it verifies those updates rather than
  pre-dates them. These two files go stale precisely because nothing
  else forces them — no task references them, no test fails when they
  lag — so the check lives here at the merge gate, and as a standing
  final task in `tasks.md` (see the tasks template). README changes
  describing the spec's behavior ride the spec branch; `ROADMAP.md`
  commits straight to `main` per the rule below.
- Repo-wide files (`CLAUDE.md`, `ROADMAP.md`, `DECISIONS.md`) commit
  straight to `main`. Spec-specific files commit to that spec's branch
  and ride into `main` only when the spec merges. Getting this backwards
  is an easy, low-stakes mistake — worth a standing rule so it isn't
  re-litigated every time.
- Keep AI co-authorship attribution on commits. It's accurate, and for a
  project meant to demonstrate this workflow, the transparency is worth
  more than a clean-looking log.
- Never force-push.

## Review cadence: tiered by risk, not uniform

Reviewing every single task at the same depth is both exhausting and
miscalibrated — a wrong CRUD field is a five-minute fix; a wrong data
model decision discovered three weeks later is not. Tier the review
cadence by how expensive a mistake would be to unwind, not by a flat
rule:

- **Review after every phase**, with a phase bundle — the default
  everywhere, foundational phases included. A foundational phase is
  short by construction, so its review comes a day later rather than
  the same afternoon, and the pre-merge sweep is still behind it.
- **Per-task review only where the planner marks it** — a task whose
  mistake would be genuinely expensive to unwind (a data-model contract
  a dozen later files will depend on), flagged in `tasks.md` with
  `review: per-task`. The exception, not a phase-wide rule.
- Re-tighten around anything that turns out to be a genuine judgment
  call, even mid-phase, rather than treating the cadence as fixed once
  set.

This is the *reviewer's* cadence, and it used to be heavier: per task
throughout foundational phases, a rule from when the person's attention
was the scarce resource and a per-task look at foundational work cost
nothing else. With a token-priced reviewer it did — on the first specs
measured, review cost about twice the implementation it reviewed. At
the product-owner level every one of these reviews is a
skeptical-reviewer pass, and none of them pauses for the person. The
person's own pauses follow a different rule: after each phase (unless
they've said to run further), and whenever something unexpected
surfaces that bears on spec adherence. At the technical-lead level the
per-task reviews the planner marks are the person's as well. Either
way, reviews are scoped to a bundle — the diff, the plan sections it
implements, the acceptance criteria it serves — never a fresh
whole-codebase read; see "Keeping reviews cheap" in
`references/collaboration-workflow.md`.

## Model tiering: decisions at the top tier, execution one tier down

The best available model does the things that are genuinely
decisions — the spec conversation, plan and task drafting (the
`sdd-planner`, one dispatch per spec), and the skeptical-reviewer on
sign-off and on routine-but-real decision reviews — and nothing else.
The orchestrating session itself runs one tier down, at medium effort:
it takes thousands of bookkeeping turns and re-sends its whole context
on each, and on the first measured specs that re-send volume was eight
to nine times the implementers' and the dominant cost of the
workflow. The top tier reaches it only through explicit per-call
overrides on the planner and sign-off dispatches; the agent definitions
carry `effort: high` so reasoning stays full-strength inside them. The
reviewer's per-phase checks and marked per-task checks — a diff
against the plan sections
it implements — run one tier down, like the implementation they check.
Implementation itself — the edit, build, test loop that accounts for
most of a spec's tokens — runs one tier down, in the `sdd-implementer`
subagent (`assets/sdd-implementer.md`), one task per dispatch. The
split is by *role*, decided per task at execution time, not by a table
written in advance.

That distinction is the whole reason this works where an earlier
attempt didn't. The reference project tried tiering model and effort
per task, predicted up front by the foundational-vs-mechanical split,
and abandoned it: several tasks assumed safely mechanical benefited
from the top tier in ways nobody saw coming. What's different now is
that the strong model reads every task before dispatching it (Step 1
triage), reads every report that comes back, re-runs verification
itself, and has the reviewer on foundational tasks — and the
implementer is under a standing rule to stop and return the moment it
hits a judgment call rather than resolve it. The failure that sank the
static table — a lighter model quietly doing a worse job on a task that
looked mechanical — now has three independent catches instead of none.

How the loop runs, per task, in the orchestrating session:

1. **Triage** (Step 1 of the collaboration workflow). Routine →
   dispatch. Not routine → Plan Mode and the reviewer first, at the top
   tier, until what remains is transcription; then dispatch that.
2. **Dispatch with a packet, not a pointer.** The subagent starts cold
   — it sees `CLAUDE.md`, its own definition, and the prompt. Name the
   task line, the `plan.md` section it implements, the `spec.md`
   acceptance criteria it serves, the files involved, and the existing
   file whose pattern to copy. Findings from earlier tasks that aren't
   yet written down go in the packet too — or better, get written down
   first.
3. **Verify by running the verification command, not by reading.**
   The constitution names one filtered build-and-test command; the
   implementer runs it and reports its output verbatim. For a task the
   planner marked `review: per-task`, the orchestrator re-runs it, then
   assembles a review bundle with shell (diff, task line, plan section,
   acceptance criteria) and invokes the reviewer on that alone; for
   every other task the implementer's output is the verification and
   the phase review is the check. Open the diff yourself only when
   something failed. If the orchestrator reads every diff in
   full, the work has been paid for twice.
4. **One review, at most one re-review, per task.** The re-review sees
   the findings and the fix diff, nothing more, and whatever is still
   open after it goes to the tier log and the pre-merge sweep. Blocking
   is defined narrowly — would fail an acceptance criterion or a test,
   or contradicts the plan or constitution — and nothing else blocks.
5. **Commit, check the box, record findings — and nothing else by
   hand.** The orchestrator is the only writer of `tasks.md` and the
   only one who commits; a commit means orchestrator-verified. Findings
   from the report go into `plan.md` or `tasks.md` now, not later,
   since the next implementer won't have seen them otherwise. The
   orchestrator does not implement the reviewer's second-look notes
   itself, and does not do device, browser, or visual verification by
   hand: second-look items go to the log or the next task's bundle, and
   visual checks are the implementer's Verify criterion or the person's
   attestation at the phase pause. On the first measured specs, "folded
   in by the orchestrator" and "seen by the orchestrator" were
   recurring log entries — each one the orchestrating session doing
   work the tiering exists to move off it.
6. **Sequential, one task at a time, and drop the carried context at
   each phase boundary.** Commit-per-task and shared files make
   parallel implementers messy; parallel dispatch is a deliberate
   opt-in for a later day, not the default. At a phase pause, `/clear`
   and resume from the first unchecked task; compact only mid-phase if
   the context grows large. The orchestrator's context is re-sent on
   every turn, and it must not carry the whole spec.

**The escape hatch.** If the implementer fails verification twice on
the same task, or returns "stopped on a judgment call" for something
the orchestrator considers well-specified, the orchestrator does that
task itself and notes the miss in `tasks.md`. That's
the surviving form of the reference project's lesson: a tier assignment
is a guess to verify, and the misses are the data.

**A third tier is available but off by default.** The dispatch can
override the implementer's model per call — Sonnet for a task that
meets all three of: an existing automated check as its Verify criterion
(a manual-check task never drops tiers, because the orchestrator can't
cheaply verify it), a named file in the codebase whose pattern it
copies, and a small footprint. Leave it off until a project's first
spec under this policy shows Opus dispatch working, then turn it on in
that project's `CLAUDE.md` if the numbers justify it.

**Measure it — the first measurement went the wrong way.** The
subagent's return reports its token usage; log it per invocation in
`tasks.md`'s tier log, alongside any escape-hatch misses, and compare
the spec's total (from `ccusage session` afterward — the orchestrator
can't see its own usage) against a previous spec of similar size. The
first specs measured under this policy cost *more* than the
single-session regime, not less, for reasons that are now rules above:
an unbounded fix-and-re-review loop in which a cold reviewer found a
new objection every round (over half of one spec's subagent tokens sat
in four tasks' loops); implementers and the orchestrator ingesting raw
build logs; the orchestrator re-running every verification at the top
tier; a whole-codebase sweep at the top tier; and one orchestrator
session accumulating the entire spec. The loop cap, the bundles, the
verification command, the sweep bound, and the per-phase session are
the response. If a spec measured under those still loses to the
single-session regime on the top tier's budget, roll the implementer
layer back and keep only the reviewer changes: the structural argument
for dispatch — cheaper rates on the bulk of the work, small fresh
contexts — holds only while the coordination overhead stays smaller
than what it replaces.

**Two names, one place.** The constitution's model policy names the
top tier and the step-down tier once; everything else refers to the
roles. The session's model and effort live in the project's
`.claude/settings.json`, written at setup from
`assets/settings-template.json` and recreated by the orchestrator if
missing — project settings outrank the app's picker for new sessions,
so they hold without anyone remembering. The fallback, when the top
tier's budget is exhausted: drop the override on the planner and
sign-off dispatches for the rest of the window, and log what ran.

**Spec conversations: chat at the project's start, Claude Code
after.** The idea conversation, the constitution, and the first spec
happen in chat — there is no codebase yet, and chat is the top tier at
the person's own setting. Every later spec conversation happens in
Claude Code, in a session of its own that ends — with a new session,
not `/clear`, since a clear keeps the model — when the spec is
approved; never inside an orchestrating session, whose context is the
cost the tiering exists to contain. One constraint the skill can't
design around: a session has one model, set at start, and this
project's default is the step-down tier. So a spec session opens by
stating which model it's running, and if that's the step-down tier, it
asks the person to pick the top tier for this session only before the
conversation continues. That is the single picker choice in the whole
workflow — and it's a choice about where the person's own thinking
runs, not bookkeeping, which is why it's the one left to them.

**Cache re-sends are the cost, so context size and turn count are the
levers.** On the measured sessions, cache reads were 97% of all
tokens. Clear at every phase boundary and at spec end — the skill
resumes cold from `tasks.md` for free; compact mid-phase only if the
context grows large; never clear mid-task. Keep spec conversations in
their own session, cleared afterward. Batch bookkeeping into single
shell commands. Each turn saved is a re-send of the whole context
saved.

The policy is written into each project's `CLAUDE.md` (see the
constitution template's "Model policy" section), next to the
involvement level. The two are orthogonal — a product owner never sees
any of this — but both are decide-once-at-the-start settings, and they
belong together. The skeptical-reviewer's definition defaults to one
tier down (`model: opus`), the right tier for its frequent per-task and
per-phase checks and for the pre-merge sweep; the orchestrator
overrides it up for sign-off and decision reviews only. See "Keeping
reviews cheap" in the collaboration workflow for the reasoning, and for
the bundles that keep every review — and every implementer dispatch —
from reading the codebase at all.

## The flow at a glance: where each step runs, and on what

Everything above, laid out as the sequence a spec actually follows.
"Fable" and "Opus" here stand for the top tier and the step-down tier
named in the project's `CLAUDE.md` model policy; the roles are what's
fixed, the names change as models do.

**A brand-new project, once.** Nothing has a codebase yet, so nothing
needs Claude Code until implementation:

| Step | Where | Model | Who's talking |
|---|---|---|---|
| Idea conversation | Chat | Fable | the person and Claude |
| Constitution → `CLAUDE.md` + `.claude/settings.json` | Chat, then committed | Fable | the person and Claude |
| First spec → `spec.md` | Chat | Fable | the person and Claude |
| First plan and tasks | Chat | Fable | Claude drafts; sign-off per involvement level |
| Implementation | Claude Code | Opus session; Opus implementers | orchestrator |

**Every spec after that.** The project's `.claude/settings.json` opens
every Claude Code session on Opus at medium effort; the agent
definitions and the orchestrator's overrides do the rest:

| Step | Where | Model | Who's talking |
|---|---|---|---|
| Spec conversation → `spec.md` | Claude Code, **a session of its own** | Fable — the session opens on Opus, says so, and the person switches to Fable for this session (the model selector, or `/model fable`); effort follows the model from settings | the person and Claude |
| Spec approved | **new session** — not `/clear`, which keeps the session's model | — | — |
| Plan and tasks drafted | Claude Code, new session | Opus session dispatches `sdd-planner` at **Fable, high** | orchestrator → planner |
| Sign-off | same session | `skeptical-reviewer` at **Fable, high**; one review, at most one re-review | orchestrator → reviewer |
| Spec-conformance summary | same session | Opus | orchestrator → the person |
| Implementation, per task | same session | `sdd-implementer` at **Opus, high**, on a task bundle | orchestrator → implementer |
| Marked per-task review | same session | `skeptical-reviewer` at **Opus, high** | orchestrator → reviewer |
| Phase review | same session | `skeptical-reviewer` at **Opus, high**, on a phase bundle | orchestrator → reviewer |
| Phase pause report | same session | Opus | orchestrator → the person, who attests by using the app |
| Phase boundary | `/clear`, new session | Opus, medium | — |
| Pre-merge sweep | last phase's session | `skeptical-reviewer` at **Opus, high**, documents + spec diff | orchestrator → reviewer |
| Close-out and merge | same session | Opus | orchestrator |

**The one manual step** is the model switch at the top of each spec
session. The session prompts for it; it can't be automated, because a
session has exactly one model and the project default is the step-down
tier. Everything else resolves from `.claude/settings.json`, the agent
frontmatter, and the orchestrator's overrides. Note the asymmetry:
`/clear` resets context but keeps the session's model, which is right
at a phase boundary (the session is already on the step-down tier) and
wrong after a spec session (it would leave planning and orchestration
on the top tier) — so a spec session ends with a new session, not a
clear.

**Fable's footprint per spec** is the spec conversation, one planner
run, one sign-off (plus at most one re-review), and any routine-but-real
decision reviews. Everything that has a turn count runs on Opus.

## Principles worth generalizing

These aren't language-specific or platform-specific — they're patterns
that showed up repeatedly enough across one real build to be worth
carrying into the next one from day one, instead of rediscovering each
time.

1. **Test the architectural claim, don't just assert it in a document.**
   If a plan document says "this schema is compatible with X" or "these
   colors are distinguishable," that's a testable claim — write the test
   that would catch it being false, don't just write the sentence and
   trust it. This generalizes further than it sounds: the same instinct
   caught a database-compatibility bug, a color-contrast bug, and an
   asset-loading bug in one project, because it's really "any claim
   about how the system behaves is a test, not a comment."

2. **A passing test is not evidence it can fail.** Mutation-test
   anything that matters: deliberately break the rule the test claims to
   guard, and confirm the test actually goes red. Tests that can't fail
   are surprisingly common and surprisingly hard to spot by reading them
   — they read exactly like real coverage. When one turns out to be
   false-passing, audit for the same *shape* elsewhere rather than
   fixing only the instance found.

3. **Verify the mechanism, not a proxy for it.** When checking whether
   something actually happened, instrument the thing itself — a log
   statement, a direct check — rather than inspecting a visual or
   indirect artifact that might not reliably show it. A fast or
   synchronous action can complete before any screenshot catches it;
   "I didn't see evidence of X" and "X didn't happen" look identical and
   mean opposite things. This mistake is expensive specifically because
   it produces confident, wrong conclusions rather than uncertainty. A
   corollary worth remembering: a long-standing, well-documented platform
   capability is the least likely thing in the room to be broken —
   suspect the newest, most custom code first, and check a surprising
   "this doesn't work" conclusion against real documentation before
   accepting it.

4. **One source of truth, not two things that could silently drift.**
   Any time the same fact, threshold, or calculation is needed in two
   places, make one canonical and have the other reference it. Two
   independently-written versions that happen to agree today are a bug
   waiting for the day someone edits only one of them.

5. **Correctness over reference-fidelity, but never silently.** When an
   implementation and a design reference (or a spec and an earlier
   assumption) genuinely conflict, resolve toward whichever is actually
   correct — but always surface the divergence explicitly, with
   reasoning, rather than quietly picking a side. The person steering
   the project should see every place execution disagreed with the plan,
   not just the places it matched.

6. **Reserve real "use it yourself" time — don't review only diffs and
   summaries.** Some of the most important catches in a build like this
   come from someone actually using the running thing, not from reading
   what changed. A summary can describe a feature working correctly
   while the actual feel of it is off in a way no diff would show.

7. **Own mistakes plainly, in both directions.** This applies to the AI
   and the human equally. When a wrong technical conclusion gets
   reached, say so plainly and explain what the right verification
   would have been — don't quietly correct course without naming the
   error. When a person's own edit or assumption caused a problem
   (a stale file handed over as current, a misplaced instruction), that
   deserves the same treatment, not defensiveness.

## The multi-writer file problem

The moment implementation starts, `tasks.md` gets written by more than
one party — the planning conversation adds scope and reshuffles tasks,
and the implementer checks boxes and adds findings as it works. This is
different in kind from every other document, which has exactly one
writer, and it needs different handling:

- **Never edit from memory or an old copy.** Before making any change,
  get the actual current file and verify it, even if a version was seen
  five minutes ago in the same conversation.
- **Prefer small, targeted edits over regenerating the whole file.** A
  full-file replacement silently discards whatever the other party
  added since the copy being edited from was taken — checked-off boxes,
  new findings, added scope. This is the single most common way this
  kind of project gets corrupted, and it's avoidable by discipline alone.
- **After any edit, sanity-check structure**, not just content — grep
  for section headers and ID sequences to confirm nothing got duplicated
  or dropped. This catches an editing mistake immediately rather than
  three tasks later.

## Session and context hygiene

- In Claude Code, `/clear` at every phase boundary and at spec end —
  not `/compact` — because a project with real documentation
  discipline loses almost nothing when the conversation resets:
  `CLAUDE.md` re-reads automatically, and `tasks.md` is exactly the
  file designed to answer "where was I" cold. Measured across real
  sessions, cache re-sends of carried context were 97% of all tokens,
  so the carried context *is* the cost. `/compact` is for staying
  mid-phase when the context has grown large; never clear mid-task.
- In a chat interface without that command, the equivalent move is
  starting a fresh conversation with the project's key documents
  uploaded to its knowledge base — same principle, same payoff, since
  the real state was never only in the chat to begin with.
- If a long planning conversation accumulates genuinely valuable
  reasoning that isn't fully captured in the terse final form of the
  docs — the *why* behind a decision, not just the decision — consider
  writing a short session summary before moving to a fresh conversation,
  so that reasoning isn't lost even though it's not literally in the repo.

## The collaboration workflow

See `references/collaboration-workflow.md` for the full, step-by-step
version of this. In short: **the default is to stay inside Claude Code**,
using Plan Mode (research and propose before touching any files) for
real decisions, and a custom reviewer subagent (see
`assets/skeptical-reviewer.md`) for a genuinely independent second look
without leaving the tool. A separate chat conversation is reserved for
two specific triggers, not general "foundational" judgment: something in
the design turning out infeasible or needing substantial rework, or a
previously-unknown consideration surfacing that would materially change
the project's direction. Everything else — including plenty of things
that feel weighty in the moment — resolves inside Claude Code. This
tiering follows the same risk-based logic as review cadence above,
applied to *which surface a decision happens on*.

## A realistic first sequence for a new project

Not every step below deserves equal engagement. For a solo or personal
project specifically, the natural weighting is uneven: the idea itself,
the spec's user flows, and the design direction are where iteration
genuinely pays off. The constitution's technical choices — framework,
hosting, package manager, and similar — are comparatively fungible;
"good enough, quickly" costs little there, and treating them as an
extended negotiation is a mismatch of effort to what's actually at
stake. This is a preference, not a universal rule — say so explicitly
if a given project actually wants more rigor upfront on the technical
side (real infra stakes, a team involved), and follow that instead.

0. **Idea conversation, before any technical decision.** Audience,
   purpose, what makes this distinctive, the core loop or the point of
   the thing. Reaching for a framework choice before the idea itself is
   settled is working backwards — technical decisions usually clarify
   naturally once the idea is clear, not the other way around.
1. **Constitution conversation.** Platform/language/architecture choices,
   testing philosophy, dependency policy, and the person's involvement
   level (see "Involvement level" above — ask once, directly, and
   default to product owner) — write `CLAUDE.md` before any code
   exists, so the first thing Claude Code reads when it scaffolds the
   project is the constitution, not its own defaults. The same step
   writes `.claude/settings.json` from `assets/settings-template.json`
   — the session model and effort the model policy relies on — and
   commits it with the constitution. The person never creates this by
   hand; it's part of scaffolding, and the orchestrator recreates it
   if it's ever missing. Move through
   this efficiently once the idea is settled: when someone doesn't have
   a strong preference on a technical choice, recommend a sensible
   default and explain briefly why, rather than opening it up as an
   extended exploration. "No preference" is a signal to move quickly,
   not an invitation to generate a longer list of options. If asked to
   help explore an option (hosting, for instance), give a genuine,
   opinionated recommendation grounded in what's already been decided
   — not a neutral menu that hands the decision back.
2. **First spec.** Accept that it'll be larger than specs after it (see
   "The first-spec exception"). Push on ambiguity now — it's nearly free
   to resolve in conversation and expensive to resolve after code exists.
3. **Plan.** Translate the spec into real technical design — actual
   types, actual data flow, actual file structure. This is where
   "someone should double-check this claim" moments should be written
   down as things to verify, not assumed correct.
4. **Design exploration**, if the project has a UI — see "Design
   exploration, when the project has a UI" above for what actually needs
   to be in the brief and why. The deliverables are screens exported as
   images plus a tokens document, both becoming implementation
   references for whoever builds from them.
5. **Tasks.** Ordered, small, independently verifiable — and tiered by
   risk for review cadence, per above.
6. **Implement**, with the review discipline actually followed, not just
   agreed to in principle. The first phase or two is where the pattern
   either sticks or doesn't — it's worth being strict early even if it
   feels like overkill, because that's also when a mistake is cheapest
   to catch.

**For every spec after the first, this same sequence applies minus step
1** — the constitution already exists and stays in force unless this
particular feature genuinely requires amending it, per `CLAUDE.md`'s own
rule (amend explicitly, in its own commit, before the spec proceeds).
Steps 0 and 2 still happen the same way, as a conversation with the
person — in Claude Code now, in a spec session of its own at the top
tier (see "Spec conversations" under "Model tiering") — a second or
tenth spec doesn't skip the idea-and-design phase just because the
project already has a working codebase. Steps 3 and 5, though, change
hands once the project has shipped code — see the next section.

## Who authors plan.md and tasks.md: a phase transition

Authorship of `plan.md` and `tasks.md` splits along the what/how
boundary, not the chat/Claude-Code boundary — and which tool holds the
pen for the *how* depends on whether there's a codebase yet.

**Until the project has shipped code, plan in chat.** A first spec's
plan *invents* the architecture rather than extending one — there is
nothing to inspect, and the design conversation holds all the relevant
context. This is the original shape the skill grew from, and it remains
right for that phase.

**Once shipped code is what plans extend, plan in Claude Code.** A plan
against a real codebase needs the actual model definitions, the actual
view structure, the actual dependency-injection shape — ground truth
that chat can only see through a manually-uploaded snapshot that starts
subtly stale and drifts from there. The workaround is a relay: Claude
Code prints state into chat, chat authors the plan against the paste, a
transcription layer whose only function is preserving a rule written
for a condition that no longer holds. Instead, once `spec.md` is
approved: the orchestrating session assembles a planning bundle with
shell — the spec, the previous spec's `plan.md` and `tasks.md` as the
pattern, a file listing — and dispatches the `sdd-planner` subagent
(`assets/sdd-planner.md`) on it, once, at the top tier. The planner
reads the code the spec touches, writes both files marked Draft, and
returns a summary with its token usage for the tier log. The
orchestrator commits the drafts to the spec branch with the PR still in
draft, and the skeptical-reviewer signs off. The exploration a plan
needs is the expensive part of planning, and this puts it in a
discardable context, bounded by the bundle, instead of in the session
that then carries it through every sign-off round and into
implementation.

**`spec.md` stays a conversation with the person in both phases** —
in chat for the first spec, in a dedicated Claude Code spec session
after. It captures product intent, user-facing behavior, and decisions
— the design conversation's actual job — and the model writing it
should be reasoning about the product, not reading the code.

**Who signs off depends on involvement level, and this is the one place
the levels differ materially.** At the technical-lead level, the person
reads and approves `plan.md` and `tasks.md` before any implementation
task starts, in either authorship phase — drafting relocated; approval
didn't. At the product-owner level, the planner's draft plus the
skeptical-reviewer's sign-off is the gate: the reviewer checks the
draft against `spec.md` and `CLAUDE.md`, blocking findings go back to
the orchestrator to be fixed and re-reviewed once, and the person
receives a
**spec-conformance summary** rather than the plan itself — which
acceptance criteria the plan serves and how, where it deviates from the
spec and why, and any product question it surfaced that needs their
call. They approve the *what* that summary describes; the *how* is
already signed. Supervision hasn't been removed, it's been relocated:
to the spec (the contract), the reviewer (the check), and the
escalation triggers (the exit). That's what keeps this from reading as
"Claude Code plans and builds unsupervised."

Two risks worth naming, both covered by machinery the workflow already
has. A planner with the code open may anchor on what's easy to build
over what's right — but Plan Mode separates thinking from doing, the
skeptical-reviewer exists precisely to challenge convenient answers,
and the spec, authored in conversation without implementation
anchoring, remains the contract the plan is reviewed against. And a product
decision surfacing mid-plan could get settled silently in `plan.md` —
but the escalation rule already covers this: anything that turns out to
be a product decision goes back to the person (in practice, back to
the spec chat) rather than being resolved by the planner.

## Building tasks.md: from plan to an ordered task list

Turning an approved `spec.md`/`plan.md` into `tasks.md` is a skill worth
being deliberate about, not just "break it into steps." A few concrete
things made this work well on the reference project:

1. **Every task has a checkable "Verify:" criterion**, not just a
   description of what to build. "Implement the item list" isn't a
   task; "`ItemListView`: list with filter/sort controls, using
   `ItemListViewModel`. Verify: [specific test or manual check]" is. If
   a task's completion can't be checked concretely, it's still too
   vague to hand off.
2. **Phase boundaries follow dependency, not just feature grouping.**
   Foundational work (data models, shared utilities other screens will
   reuse) comes first, in its own phase, before anything that depends
   on it. Within one feature area, view models typically precede the
   views that use them — build and test the logic layer, then build UI
   against it.
3. **Shared components get one task, reused everywhere — not rebuilt
   per screen.** If two screens need the same picker or the same
   formatting helper, that's a task in a shared-utilities phase,
   referenced by every screen's own task rather than reimplemented each
   time it comes up.
4. **The whole spec's task list gets planned before implementation
   starts**, not written phase by phase as you go. This is the same
   "approved spec and plan before implementation" discipline `CLAUDE.md`
   already states, applied to `tasks.md` too — reviewed as a whole
   before handoff, even though execution then proceeds phase by phase.
5. **When real scope gets discovered mid-build that wasn't in the
   original plan, append a sub-lettered task (`T036a`, `T036b`) rather
   than renumbering everything after it.** Cheap, doesn't disturb
   already-completed task references, and reads honestly as "found
   here" rather than implying it was planned from the start.
6. **State the review cadence per phase, not per task** — see "Review
   cadence" above — and state the person's pause cadence separately
   from the reviewer's, per the involvement level in `CLAUDE.md`.
   `tasks-template.md`'s handoff note is where both get stated
   explicitly for whoever picks up implementation, along with the shape
   of the report a pause should produce.

## Bugs found after a spec ships

A bug discovered in already-merged code isn't a new spec, and it isn't a
reason to reopen the spec branch that shipped it — that branch's job
ended at merge. Handle it on a spectrum, matching its actual size:

- **Trivial** (a typo, an off-by-one, a clearly-wrong constant, no real
  design implication) — a small dedicated branch (`fix/<short-
  description>`, not the `<NNN>-<slug>` spec convention), a clear commit
  message explaining what was wrong and why, PR, merge. No `plan.md`
  update needed — the commit message carries the explanation.
- **Real, but contained** (the root cause needed real investigation, or
  the fix corrects a genuine misunderstanding about how something
  works, even if the code change itself is small) — same small
  branch/PR shape, but also update the original spec's `plan.md` in
  place to reflect the corrected understanding. `plan.md` is a living
  document, not a frozen snapshot (see "The document set" above) — a
  fix that changes what's actually true about the system belongs there,
  even months after that spec shipped.
- **Bigger than a fix** (the correction needs substantial rework, or
  reveals a genuinely new design question) — this has become a new spec
  in its own right, not a patch. The same two triggers that already
  govern escalation during implementation apply here too: infeasibility/
  rework and a direction-changing unknown are exactly the signal that a
  "bug fix" has actually turned into something needing the real
  spec → plan → tasks treatment before more code gets written — the
  spec in its own session with the person, the plan and tasks per the
  authorship phase transition above (which, for a project with shipped
  code, means the `sdd-planner`).

The existing per-task triage (see "The collaboration workflow" and
`references/collaboration-workflow.md`) already governs how much
scrutiny any given fix needs — routine ones proceed normally, real ones
get Plan Mode and the subagent, foundational ones escalate. Nothing new
there; what's actually missing is the git shape, since "one branch per
spec" was never written with work-that-isn't-a-spec in mind.

## Using the templates

`assets/` has starting points for the four core documents —
`CLAUDE-template.md`, `spec-template.md`, `plan-template.md`, and
`tasks-template.md` — plus `design-brief-template.md` for projects with
a UI, `settings-template.json` (the project's `.claude/settings.json`,
written at setup), and three ready-to-use Claude Code subagent
definitions: `skeptical-reviewer.md`, `sdd-implementer.md`, and
`sdd-planner.md`.
The document
templates are skeletons with placeholders and inline guidance
comments, not fill-in-the-blank forms — expect to restructure sections
as the actual project's needs diverge from the template, the same way
real projects always do.
