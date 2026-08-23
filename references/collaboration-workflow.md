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
> against CLAUDE.md and the current spec/plan."

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

> "Have the skeptical-reviewer check the last commit's claims against
> what actually got tested."

Good moments to do this: anything that felt uncertain while it was being
built, and before a spec's PR comes out of draft and merges.

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
