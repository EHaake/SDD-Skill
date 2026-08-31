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

## Spec-driven workflow

This project follows spec → plan → tasks → implement, gated by human
review between each phase. Artifacts live in `specs/<NNN>-<slug>/`:

- `spec.md` — what and why, user-facing behavior, acceptance criteria,
  explicit non-goals. No implementation detail.
- `plan.md` — technical design: types, data flow, what changes where.
- `tasks.md` — ordered, small, independently verifiable tasks.

Do not begin implementation on a feature without an approved spec and
plan in that feature's directory. When resuming a session, check
`specs/<feature>/tasks.md` for current state before doing anything else.

## Collaboration workflow

If the `spec-driven-development` skill is installed
(`~/.claude/skills/spec-driven-development/` or a project-level
`.claude/skills/`), its collaboration workflow applies automatically —
routine tasks proceed normally, real decisions resolve via Plan Mode and
the `skeptical-reviewer` subagent, and the person is looped in only when
something in the design turns out infeasible or needs real rework, or a
previously-unknown consideration surfaces that would materially change
the project's direction. Nothing needs to be repeated here.

<!-- Add project-specific refinements only if this project's escalation
criteria genuinely differ from the skill's default — a different risk
tolerance, a different definition of "foundational" for this particular
codebase. Most projects won't need anything here at all; the one case
this project came from needed a hand-written version only because the
skill didn't exist yet at the time. -->

## Verification

After any implementation task, Claude Code must:

1. Build the project.
2. Run the test suite.
3. Report the actual pass/fail output, not a paraphrase.

A task is not complete until steps 1–2 are green. Do not weaken, skip, or
delete a test to make it pass — if a test seems wrong, flag it and ask.

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

- One commit per completed task where practical, referencing the task ID.
- Commit messages describe what changed and why, not "implement task 3".
