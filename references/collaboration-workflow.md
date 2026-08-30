# Collaboration Workflow: Claude Code, the Reviewer Subagent, and Chat

The default is to stay inside Claude Code. A separate chat is a
deliberate escalation for specific moments, not the normal loop. This
document is written to be followed without guessing — if a step feels
ambiguous in the moment, that ambiguity is the thing to fix in this
document afterward, not something to improvise around silently.

## One-time setup

1. Place `skeptical-reviewer.md` (in this skill's `assets/` folder) at
   `~/.claude/agents/skeptical-reviewer.md` — the user-level directory,
   so it's available in every project automatically, not just the one
   it was first set up in.
2. In any Claude Code session, confirm it's recognized: ask "what
   subagents do you have available?" or equivalent, and check
   `skeptical-reviewer` appears.
3. Done. It never needs to be recreated per project.

If a specific project wants its own customized version instead of the
shared one, a copy at that project's `.claude/agents/skeptical-reviewer.md`
takes precedence over the user-level one for that project only.

## The per-task loop

**On cadence, stated plainly up front**: the subagent isn't invoked on a
fixed schedule — not every task, not automatically at the end of every
phase. It's driven by the nature of each decision, per task (Step 1
below), which could mean zero invocations in an all-mechanical phase or
several in a decision-heavy one. Layered on top of that per-decision
trigger is one recurring checkpoint that *is* close to a hard rule:
before a spec's PR comes out of draft and merges (see "After
implementation" below) — a whole-spec consistency sweep, not tied to
any single task. Routine work should never see the subagent at all;
that's the point of Step 1 existing.

### Step 1 — Is this routine?

Before Claude Code starts on a task, ask whether it's well-specified and
mechanical — matches an established pattern already in the codebase, no
real judgment call involved. If yes: let it proceed normally, no
subagent, no separate chat. This is most tasks, most of the time, and
should stay fast.

If the task involves any of the following, it's not routine — continue
to Step 2:
- Touches the data model, a core architecture choice, or something many
  other files will end up depending on.
- A genuine design or product tradeoff with no obviously correct answer.
- A design reference (a mock, an earlier decision) conflicts with
  something else, and reconciling them isn't mechanical.
- Would be expensive to unwind if it turns out wrong — not a five-minute
  fix.

**A task can trip the surface of one of these and still correctly be
routine, if the actual judgment call was already discharged upstream** —
in an approved `plan.md` section, or in an earlier review round — and
what remains at execution is genuine transcription into code, not an
open decision. The test is whether a real, undischarged judgment call
remains at the moment of execution, not whether the topic sounds
architecturally significant. A whole phase reading as "all routine" can
be the success condition of good upfront planning, not evidence the
triage is being skipped — but that's only true if the hard parts were
genuinely settled earlier, not merely unexamined now. Worth being able
to point to *where* a given decision was actually made, not just assert
that it was.

### Step 2 — Use Plan Mode

Activate Plan Mode (`Shift+Tab` twice, or `/plan`) before letting Claude
Code touch any files. Let it research read-only and propose an approach.
Read the plan yourself. Sometimes this alone resolves it — if the plan
is obviously right, or obviously needs one small correction you can just
state, do that and move on without further escalation.

### Step 3 — Decide: subagent, or subagent-plus-chat?

Ask whether this is **routine-but-real** (a genuine decision, but Claude
Code is well-positioned to reason through it) or hits one of two
specific triggers — not general "foundational" judgment, which is easy
to over-apply:

1. An aspect of the design or a feature in `spec.md`/`plan.md` turns out
   to be infeasible, or needs substantial rework to actually build.
2. A previously-unknown consideration surfaces where deciding it either
   way would materially change the project's direction — not an
   implementation detail with a reasonable default, a genuine fork.

**Routine-but-real** (the large majority of real decisions, including
plenty that feel weighty in the moment) → invoke the subagent, resolve
within the same session:

> "Before I approve this, have the skeptical-reviewer check this plan
> against CLAUDE.md and specs/004-search/spec.md and plan.md."

The subagent starts cold: it sees the invocation prompt, CLAUDE.md,
and a git status snapshot — nothing else from the session. Name the
spec directory and the specific artifact under review in the prompt;
it can't infer them from a conversation it never saw.

Read its findings. If it flags something real, discuss with the main
Claude Code session, revise, and optionally re-review. If it comes back
clean, approve and let implementation proceed.

**One of the two triggers** → do the subagent review as well, but also
bring the specific question to a separate, fresh chat before finalizing
(see below).

### Step 4 — Escalating to a separate chat, narrowly

When a question does warrant leaving Claude Code, bring the *question*,
not the project state:

- State the decision or the disagreement plainly, in one or two
  sentences.
- Paste the specific relevant excerpt — the paragraph in question, the
  two conflicting claims, the actual code or test — not the whole file.
- Let that conversation resolve the one question in front of it.
- Take the answer back to Claude Code as a short, targeted instruction.

This is meant to be rare per project — reserved for what would already
earn the tightest review tier, not a routine step. If it's happening for
most tasks, something in Step 1's triage is being applied too
conservatively.

## After implementation, not just before

The subagent is equally useful pointed at something already built,
not only a plan:

> "Have the skeptical-reviewer check this commit's claims against what
> actually got tested. The diff is in scratch/review-input.diff."

One mechanical constraint: the reviewer has no Bash, deliberately —
its read-only design is the point — so it cannot run `git diff`,
`git show`, or `git log` itself. When a review concerns a commit or a
diff, the invoking session must supply it: paste the diff into the
invocation prompt, or write it to a file and name that file. Without
that, the reviewer can only see the current state of the files, not
the change.

Good moments to do this: anything that felt uncertain while it was being
built, and before a spec's PR comes out of draft and merges. For that
pre-merge pass, say so explicitly, so the whole-spec sweep happens on
purpose rather than as a guess about scope:

> "This is the pre-merge whole-spec sweep for specs/004-search. Have
> the skeptical-reviewer sweep spec.md, plan.md, tasks.md, ROADMAP.md,
> and DECISIONS.md for drift before this merges."

## Recording what a review decided

After acting on a review's findings, record each real decision in the
document it belongs to: a plan.md entry for a technical decision
(plan.md is a living record — this is exactly the content it exists
for), a spec.md correction if behavior was mis-stated, a CLAUDE.md
principle if the lesson generalizes, or a DECISIONS.md entry for
business or process context. Include a line on why it was resolved
that way, and reuse the review's own severity labels (blocking /
second look / solid) so the record and the reviews speak the same
language. This is what makes a discharged judgment call citable later
— Step 1's triage can point at where a decision was actually made
instead of just asserting that it was.

Record findings and resolutions, never coverage. A "reviewed on this
date, found clean" marker is a suppression list waiting to happen: the
sweep part of a review is drift detection, and any later change can
put two documents that agreed at the last review into contradiction —
a past clean verdict says nothing about the present. For the same
reason, never paste a previous review's verdict into a new reviewer
invocation. A fresh reviewer that starts from its predecessor's
agreement is anchored on exactly the failure mode it exists to catch;
give it the current documents and the current question, nothing else.

The main session does this writing, not the reviewer — the reviewer
has no write access, and that's part of its design, not a gap to work
around.

## When multiple findings surface at once

A subagent review, or a spec close-out pass, often surfaces several
things together — some routine-but-real, some genuinely needing the
person's own attestation, some hitting an actual escalation trigger.
Don't bundle all of it into one "come back and sort through this"
conversation just because it surfaced at the same time. Before bringing
anything to the person:

- Resolve every routine-but-real item first, inside Claude Code, via
  Plan Mode and the subagent — same as any other task, regardless of
  whether it happened to surface alongside something that does need
  the person.
- Separate what's left into two categories: things only the person can
  attest to (behavior they'd have to actually use the app to confirm —
  not determinable from the test suite or a diff), and things that hit
  one of the two escalation triggers.
- Bring only those two categories to chat, and say plainly which is
  which. A person's attention during an attestation pass shouldn't also
  get spent re-litigating a design-polish call that could have been
  settled without them.

## What "good" looks like over time

This workflow is working if: routine tasks move through Claude Code
without ever surfacing here: the subagent gets invoked periodically, not
constantly, and mostly comes back either clean or with something
genuinely worth acting on; and a separate chat gets used rarely enough
that each instance is memorable, not routine. If any of those stop being
true, that's worth revisiting — including reconsidering the triage in
Step 1, which is a judgment call that should improve with actual
practice, not something to treat as fixed on first use.
