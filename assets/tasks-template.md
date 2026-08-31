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
whoever's steering adds scope and reshuffles tasks; the implementer
checks boxes and adds findings. Never edit this file from a stale copy.
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
right after it) is where per-task review is worth the friction. -->

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

Once this file is reviewed and approved, hand it to the implementer with
something like:

> Read [constitution file] and [spec/plan/tasks paths], then begin
> implementing starting at the first task. For [foundational phases],
> stop for review after each individual task. From [phase N] onward,
> stop after each phase instead of after each task.

## Model and effort per phase (optional — skip by default)

<!-- Skip this section unless there's a genuine, active budget or quota
constraint. The reference project found that defaulting to the best
available model at maximum effort for everything was simpler and more
reliable than trying to predict in advance which tasks could safely use
less — several tasks assumed to be safely mechanical benefited from the
top tier in ways that weren't obvious until after the fact. If real
resource pressure does show up, this table is still a reasonable
starting shape — but treat any specific tier assignment in it as a
guess to verify, not a rule to trust. -->

| Phase | Model / effort | Why |
|---|---|---|
| 0 — [...] | [top tier], [high effort] | Foundational; mistakes are expensive to unwind. |
| N — [...] | [lighter tier], [lower effort] | Well-specified, mechanical. |
