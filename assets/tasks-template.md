# Tasks: [Feature Name]

**Status**: Draft — pending review
**Implements**: plan.md in this directory

<!-- Before filling this in: the skill's "Building tasks.md" section has
the actual methodology — the "Verify:" criterion pattern, dependency-
ordered phases, when to sub-letter a task instead of renumbering. This
skeleton shows the shape; that section explains how to fill it well. -->

Ordered, small, independently verifiable. Each task should be completable
(and testable) on its own. If a session ends mid-list, resume by finding
the first unchecked task — don't re-verify everything above it unless
something looks off.

<!-- WARNING, and worth leaving this comment in the real file: once
implementation starts, this file gets written by more than one party —
whoever's steering adds scope and reshuffles tasks; the implementing
session (the orchestrator — never the sdd-implementer subagent) checks
boxes and adds findings. Never edit this file from a stale copy.
Prefer small, targeted edits over regenerating it wholesale — a full
replacement silently discards whatever the other party added since your
copy was taken. This is the single most common way a project like this
gets corrupted, and it's entirely avoidable by discipline. -->

Per the constitution: every implementation task ends with an actual build
and, where tests exist for what changed, an actual test run — reported,
not summarized.

---

## Phase 0 — Project scaffolding (one-time)

<!-- Foundational setup. Mistakes here are cheap to catch immediately
and expensive to unwind later — this phase (and the data-model phase
right after it) is where a per-task skeptical-reviewer pass is worth
the cost, and, at the technical-lead level only, a per-task pause for
the person. -->

- [ ] **T001** — [...] *Verify: [concrete, checkable outcome].*

## Phase 1 — [Data model / core architecture]

<!-- Still foundational. Same tight review cadence as Phase 0. -->

## Phase 2 onward — [feature work, roughly in dependency order]

<!-- Once the foundation is solid, review cadence can relax to per-phase
rather than per-task — a wrong view or a wrong CRUD field is cheap to
fix after the fact. Re-tighten around anything that turns out to be a
genuine judgment call, even mid-phase. -->

## Final phase — Spec close-out

<!-- Keep this phase in every real tasks.md. Merge is gated on every
task being checked, which makes this checkbox the enforcement mechanism
for updates nothing else forces — no other task references these files,
and no test fails when they go stale. -->

- [ ] **T0XX** — Update `ROADMAP.md` (drop or annotate what this spec
  shipped; add any follow-ups it surfaced) and the repo README if
  user-facing behavior or setup changed. Then request the pre-merge
  whole-spec sweep. *Verify: ROADMAP.md no longer lists this spec's
  work as future; README matches actual behavior; the sweep came back
  clean or its findings were resolved.*

---

## Handoff note

Once this file is signed off (per the involvement level in the
constitution), hand it to the implementer with something like:

> Read [constitution file] and [spec/plan/tasks paths], then begin
> implementing starting at the first task. Involvement level is
> [product owner / technical lead]. Dispatch each routine task to the
> sdd-implementer per the constitution's model policy; verify by
> running, then commit. For [foundational phases], have the
> skeptical-reviewer review after each task, scoped to that task's diff
> and the plan section it implements[; technical lead only: and stop
> for my review after each task as well]. From [phase N] onward, review
> after each phase instead. Pause for me after each phase [or: "run
> through phases X–Y without pausing"], and whenever something
> unexpected bears on spec adherence.

Every pause produces a report in this shape, in this order. The person
may not be technical, and the report exists so they can act, not so
the work is documented:

1. **Why this pause** — a phase boundary, a spec-adherence question, or
   an escalation trigger. One line.
2. **What you can now do** — behavior that exists and can be tried,
   stated as a user would experience it, so attestation is possible.
3. **Where execution deviated from the spec, and why** — every place,
   per the "never silently" principle, not just the interesting ones.
4. **What needs your decision** — product questions only. Technical
   detail lives in plan.md and the commit log for anyone who wants it;
   it doesn't lead the report.

## Tier log (recommended for the first spec under a model policy)

<!-- The constitution's model policy decides which tier runs each task
at dispatch time — there's no per-phase table to fill in. What's worth
recording here is the evidence: token usage from each subagent return
— implementer runs and reviewer invocations alike — any escape-hatch
miss (a task the orchestrator had
to redo at the top tier, and why), and, if the third tier is on, which
tasks it took and whether they held up. Compare the spec's total
against a previous spec of similar size before treating the policy as
settled. Drop this section once a project has that answer. -->

| Task / invocation | Tier | Tokens | Outcome / miss reason |
|---|---|---|---|
| T001 | [opus] | [...] | verified first try |
| T001 review | [opus] | [...] | signed off; scope statement matched the bundle |
| plan sign-off | [top tier] | [...] | fix and re-review ×1, then signed off |
