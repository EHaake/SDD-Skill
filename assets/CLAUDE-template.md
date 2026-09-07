# Project Constitution

This file is the standing contract for how this codebase is built. It loads
into every Claude Code session automatically. Specs and plans must not
contradict it; if a spec needs to, the constitution gets amended first,
explicitly, in its own commit.

## What this project is

<!-- One paragraph. What it is, who it's for, what the core loop is.
Write this once the first spec conversation has actually happened —
don't guess at it before then. -->

## Platform

<!-- Target platform/version, language, and any "no X unless Y" rules,
each with a one-line reason. Example shape:

- **Target**: [platform + minimum version]. [Back-compat policy.]
- **[UI framework, if any]**: [framework] only. No [alternative] except
  where [specific, narrow exception] — flagged when it happens, not a
  default.
- **Language**: [language + version]. [Concurrency/typing mode and why.] -->

## Architecture

<!-- Pattern (MVVM, MVC, whatever), and the actual rules that make it
real rather than aspirational — what layer owns what, what's forbidden
from importing what, how dependencies get injected for testability. -->

## Testing

<!-- Framework, and the real requirements:
- What needs tests, and what the fake/mock boundary is.
- "A task is not done until tests exist and pass — run the command
  yourself, don't assert completion from reading the code."
- Leave room here to add principles as they're discovered — this
  section should grow over the project's life, not stay static. See
  the parent skill's "Principles worth generalizing" section for the
  kind of thing that belongs here once found. -->

## Dependencies

<!-- Default policy (e.g. "no third-party packages without discussion")
and why. -->

## Project file safety

<!-- If there's a project-format file that agents reliably mishandle
(Xcode's .pbxproj is the classic example — a semi-binary format that's
easy to corrupt with a direct edit), name it explicitly and state the
safe path around it. -->

## Involvement level

<!-- Pick one, decided in the constitution conversation, and delete the
other. The skill's default is product owner. -->

**Product owner.** The person owns `spec.md`, attests to behavior by
using the app at phase pauses, and decides escalations. They do not
approve technical work: `plan.md` and `tasks.md` are drafted by the
`sdd-planner` and signed off by the `skeptical-reviewer`, each phase
(and any task the planner marked for its own review) is reviewed by
the `skeptical-reviewer` rather than the person, and what reaches the
person is a spec-conformance summary, not an architecture review.
Implementation pauses after each phase unless the person says to run
further, and whenever something unexpected bears on spec adherence.

**Technical lead.** As above, but the person also reads and approves
`plan.md` and `tasks.md`, and implementation pauses for their review
after every task the planner marked `review: per-task`.

## Model policy

<!-- Decided once, alongside the involvement level. Adjust the tier
names as models change; the roles don't. -->

- **Tiers by name**: top tier `fable`; step-down `opus`. These two
  names are the only place a model is spelled out; everything below
  refers to them.
- **The session runs at the step-down tier, at medium effort**, set in
  this repo's `.claude/settings.json` (`"model": "opus"`,
  `"effortLevel": "medium"`) so no one has to remember it. The
  orchestrating session takes thousands of bookkeeping turns and
  re-sends its whole context on each one; measured across the first
  specs, that re-send volume was eight to nine times the implementers'
  and was the dominant cost of the entire workflow. It doesn't need
  the top tier or deep reasoning to assemble a bundle and tick a box.
- **The top tier runs only inside the decisions**: the `sdd-planner`
  (one dispatch per spec) and the `skeptical-reviewer` on plan/tasks
  sign-off and on routine-but-real decision reviews — each dispatched
  with an explicit per-call override to the top tier's name. The three
  agent definitions carry `effort: high`, which overrides the session's
  medium, so reasoning stays at full strength where it matters.
- **Spec conversations happen in their own session**, cleared
  afterward (or in chat, which has no codebase to carry at all).
- **The `skeptical-reviewer` runs one tier down by default** (its
  definition says `opus`) for per-phase reviews, the per-task reviews
  the planner marks, and the pre-merge sweep. Each
  review gets a single bundle file assembled with shell — diff, task
  lines, plan sections, acceptance criteria; for the sweep, the
  documents and the spec's full diff — and reads nothing else.
- **Review loop cap**: one review and at most one re-review per
  invocation — task, phase, sign-off, or sweep. The re-review sees the
  findings and the fix diff only. Blocking
  means it would fail an acceptance criterion or a test, or contradicts
  `plan.md` or `CLAUDE.md`; nothing else blocks. Anything open after
  the re-review goes to the tier log and the sweep.
- **Implementation runs one tier down**, in the `sdd-implementer`
  subagent, one task per dispatch, sequentially. The orchestrating
  session triages each task, dispatches routine ones on a task bundle
  assembled with shell (task line, plan section, acceptance criteria,
  files, the pattern file to copy), and on return verifies with the
  verification command below — re-run by the orchestrator for tasks
  marked `review: per-task`, taken from the implementer's verbatim
  output otherwise — never by re-reading the diff. Only the
  orchestrator edits `tasks.md` or commits, and the orchestrator never
  implements second-look notes or does device or browser checks by
  hand.
- **Clear at every phase boundary and at spec end** (`/clear`, resuming
  from the first unchecked task). Cache re-sends are context size times
  turn count; a phase boundary is where the carried context has the
  least remaining value. Compact mid-phase only if the context grows
  large; never clear mid-task.
- **Batch the bookkeeping**: commit, checkbox, and tier-log row in one
  shell command; bundle assembly and dispatch back to back. Every turn
  saved is one fewer re-send of the whole context.
- **Fallback**: if the top tier's usage budget runs out, dispatch the
  planner and sign-off at the step-down tier for the rest of the
  window (drop the override). Nothing else changes; the tier log
  records what ran.
- **Escape hatch**: two failed verifications on one task, or a "stopped
  on a judgment call" the orchestrator considers well-specified, and
  the orchestrator does that task itself, noting the
  miss in `tasks.md`.
- **Third tier**: off. <!-- Turn on per project once the first spec's
  tier log justifies it: "Sonnet for tasks with an automated Verify
  check, a named pattern file, and a small footprint." -->
- **Log token usage per implementer run and per reviewer invocation**,
  plus tier misses, in `tasks.md`'s tier log for the first spec under
  this policy, and compare against a previous spec before treating the
  policy as settled.

## Spec-driven workflow

This project follows spec → plan → tasks → implement, gated by review
between each phase — the person's or the `skeptical-reviewer`'s, per
the involvement level above. Artifacts live in `specs/<NNN>-<slug>/`:

- `spec.md` — what and why, user-facing behavior, acceptance criteria,
  explicit non-goals. No implementation detail.
- `plan.md` — technical design: types, data flow, what changes where.
- `tasks.md` — ordered, small, independently verifiable tasks.

Authorship: `spec.md` is written in the chat design conversation.
Until this project has shipped code, `plan.md` and `tasks.md` are too;
once shipped code is what plans extend, the `sdd-planner` subagent
drafts them instead — at the top tier, from a planning bundle, against
the actual codebase — and the orchestrator commits them to the spec
branch with the PR still in draft. Both are signed off before any
implementation task starts: at the product-owner level by the
`skeptical-reviewer` (blocking findings fixed and re-reviewed), with
the person receiving a spec-conformance summary to approve; at the
technical-lead level by the person directly.

Do not begin implementation on a feature without an approved spec and
plan in that feature's directory. When resuming a session, check
`specs/<feature>/tasks.md` for current state before doing anything else.

## Collaboration workflow

If the `spec-driven-development` skill is installed
(`~/.claude/skills/spec-driven-development/` or a project-level
`.claude/skills/`), its collaboration workflow applies automatically —
routine tasks proceed normally, real decisions resolve via Plan Mode and
the `skeptical-reviewer` subagent, and beyond the pauses their
involvement level defines, the person is looped in only when something
in the design turns out infeasible or needs real rework, or a
previously-unknown consideration surfaces that would materially change
the project's direction. Nothing needs to be repeated here.

<!-- Add project-specific refinements only if this project's escalation
criteria genuinely differ from the skill's default — a different risk
tolerance, a different definition of "foundational" for this particular
codebase. Most projects won't need anything here at all; the one case
this project came from needed a hand-written version only because the
skill didn't exist yet at the time. -->

## Verification

The verification command for this project is:

    [verification command]

<!-- One exact command, with its output filter, that builds, runs the
tests, and prints a short summary — `xcodebuild ... -quiet 2>&1 | tail
-n 40`, `npm test 2>&1 | tail -n 40`, or a script in the repo. Raw
build logs are the largest single thing an agent can put in its
context, and in the first measured spec they were inside every
implementer, reviewer re-run, and orchestrator check. Every one of
those uses this command and nothing more verbose. -->

After any implementation task, Claude Code must run that command and
report its actual output, not a paraphrase.

A task is not complete until that output is green. Do not weaken, skip,
or delete a test to make it pass — if a test seems wrong, flag it and
ask. When the task was dispatched to the `sdd-implementer`, its verbatim
output is the verification; for a task marked `review: per-task` the
orchestrator re-runs the command itself before committing.

## Git conventions

- **One branch per spec, not per task or phase.**
- **Never commit implementation work directly to `main`.** All code
  changes happen on a spec branch. Repo-wide docs (`CLAUDE.md`,
  `ROADMAP.md`, `DECISIONS.md`) are the exception: they commit straight
  to `main`, while spec-specific files ride the spec branch.
- Open the PR as a draft immediately after pushing the branch, for a
  running diff. Only mark it ready and merge once every task in the
  spec's `tasks.md` is complete and verified.
- Marking ready has a close-out step, before any pre-merge review
  sweep: update `ROADMAP.md` (drop or annotate what this spec shipped,
  add follow-ups it surfaced) and the README if user-facing behavior
  or setup changed. README changes describing this spec's behavior
  ride the spec branch.
- Keep AI co-authorship attribution on commits — accurate, and worth
  keeping for a project meant to demonstrate this workflow.
- Never force-push.

## Commits

- One commit per completed task where practical, referencing the task ID
  — made by the orchestrating session after its own verification, never
  by the implementer subagent.
- Commit messages describe what changed and why, not "implement task 3".
