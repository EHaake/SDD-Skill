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

- **Claude (chat)** is the planning partner — narrowing scope, resolving
  ambiguity, arguing about a design decision, asking the question that
  saves a rewrite later. This is cheap, fast iteration for the *what
  and why* — the idea, the constitution, every `spec.md` — none of
  which needs repo access to think well. Early in a project it also
  authors `plan.md` and `tasks.md`; once the project has shipped code,
  that authorship moves to Claude Code (see "Who authors plan.md and
  tasks.md" below).
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
  shipped code exists, it also drafts `plan.md` and `tasks.md` in Plan
  Mode, since at that point the ground truth a plan extends lives where
  only it can see.

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

- **Foundational, hard-to-reverse work** (data model, core architecture,
  anything a dozen later files will depend on) — review after *every*
  task.
- **Mechanical, well-specified work** (CRUD following an established
  pattern, a view matching a provided screenshot) — review after every
  *phase* instead.
- Re-tighten around anything that turns out to be a genuine judgment
  call, even mid-phase, rather than treating the cadence as fixed once
  set.

**A related idea — tiering which model and effort level handles which
task, by this same foundational-vs-mechanical split — was tried on the
reference project and deliberately abandoned partway through**, in
favor of the best available model at maximum effort for everything.
It's intuitive on paper, but predicting in advance which tasks only
needed a lighter tier turned out to be less reliable than expected —
several tasks assumed to be safely mechanical benefited from the top
tier in ways that weren't obvious until after the fact. Given
sufficient budget or quota headroom, defaulting to the best available
tier for everything is simpler and was, in practice, the better call.
If genuine resource pressure ever does show up, the tiering instinct is
still worth reaching for — but treat any specific tier assignment as a
guess to verify, not a rule to trust blindly, given it was wrong often
enough here to be worth that caution.

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

- In Claude Code, `/clear` at natural phase boundaries is usually
  *better* than `/compact`, specifically because a project with real
  documentation discipline loses almost nothing when the conversation
  resets — `CLAUDE.md` re-reads automatically, and `tasks.md` is exactly
  the file designed to answer "where was I" cold. `/compact` is for
  staying mid-task without interrupting flow.
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
   testing philosophy, dependency policy — write `CLAUDE.md` before any
   code exists, so the first thing Claude Code reads when it scaffolds
   the project is the constitution, not its own defaults. Move through
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
Steps 0 and 2 still happen the same way, in chat — a second or tenth
spec doesn't skip the idea-and-design phase just because the project
already has a working codebase. Steps 3 and 5, though, change hands
once the project has shipped code — see the next section.

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
approved: Claude Code drafts `plan.md` and `tasks.md` in Plan Mode,
with the skeptical-reviewer applied to non-routine technical calls per
the collaboration workflow, and commits them to the spec branch with
the PR still in draft.

**`spec.md` stays in chat in both phases.** It captures product intent,
user-facing behavior, and decisions — the design conversation's actual
job — and needs no repo access to write well.

**The human review gates do not move.** Plan and tasks are both
reviewed and approved by the person before any implementation task
starts, in either phase. Drafting relocated; approval didn't — this
changes who holds the pen, not who signs.

Two risks worth naming, both covered by machinery the workflow already
has. A planner with the code open may anchor on what's easy to build
over what's right — but Plan Mode separates thinking from doing, the
skeptical-reviewer exists precisely to challenge convenient answers,
and the spec, authored in chat without implementation anchoring,
remains the contract the plan is reviewed against. And a product
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
   cadence" above. `tasks-template.md`'s handoff note is where this gets
   stated explicitly for whoever picks up implementation.

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
  spec in chat, the plan and tasks per the authorship phase transition
  above (which, for a project with shipped code, means Claude Code).

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
a UI, and `skeptical-reviewer.md`, a ready-to-use Claude Code subagent
definition. They're skeletons with placeholders and inline guidance
comments, not fill-in-the-blank forms — expect to restructure sections
as the actual project's needs diverge from the template, the same way
real projects always do.
