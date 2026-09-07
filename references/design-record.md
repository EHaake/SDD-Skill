# Design record: why the skill is shaped the way it is

The decision record for the skill's less obvious choices — the
reasoning and the history, kept out of `SKILL.md` so that what every
session loads is the process, not its archaeology. Nothing here is
needed to *apply* the skill. Read it when a choice is being revisited,
or when a project's evidence makes one look wrong.

Origin: the methodology was refined across one full project build (an
iOS app, from blank repo to a working, tested, CloudKit-synced v1) and
then adopted by several more. Nothing in the skill is specific to that
app. "The reference project" below means that first one.

---

## Why the orchestrator runs one tier down, at medium effort

### The question

If the top tier is the best model, why isn't it the orchestrator? A
better orchestrator makes fewer mistakes, and fewer mistakes are
cheaper in the long run even if each turn costs more up front. That
argument is sound wherever it applies. The policy is built so that it
applies as little as possible.

### What the measurement showed

On the first specs measured under the policy, with the top tier
orchestrating:

- Cache reads were about 97% of all tokens. Every turn re-sends the
  session's whole context, and that re-send — not the thinking, not
  the output — is where the tokens go.
- The orchestrating session is the longest-lived context in the
  workflow. Per task it takes many turns: assemble the bundle,
  dispatch, read the report, run verification, commit, tick the box,
  dispatch the review, read the verdict. Its re-send volume was eight
  to nine times the implementers' combined.
- A later spec cost more than an earlier one of similar size even
  after the implementer layer got cheaper. The orchestrator was where
  the savings were being spent.
- The top tier has its own usage allowance, and the orchestrator was
  consuming most of it — 82% of the weekly allowance at one point, on
  the role that needs it least.

Whichever model sits in the orchestrator's seat pays its rate on the
whole context, every turn, for the whole spec. The planner and the
sign-off are the opposite shape: short-lived contexts with a dense
concentration of judgment. So the tier goes where the *decisions* are
concentrated, not where the *turns* are.

### Why "fewer mistakes" mostly doesn't apply

The long-run argument holds when the orchestrator makes judgment calls
whose errors compound. The policy routes those calls elsewhere on
purpose:

- **Design decisions** go to the `sdd-planner` and to the sign-off
  review, both dispatched at the top tier, and to the person at phase
  pauses.
- **Correctness** is established by the constitution's verification
  command and by the skeptical-reviewer, not by the orchestrator's
  reading of a diff.
- **What the orchestrator has left is procedure**: build the bundle,
  follow the verdict, commit, don't edit `tasks.md` from a stale copy,
  don't do the task itself.

Procedural errors are cheap and self-revealing. A badly assembled
bundle fails verification or comes back "fix and re-review", and the
cost is one extra dispatch. It does not compound into a wrong
architecture, because the architecture was decided somewhere the top
tier did look.

So the trade is a bounded, occasional re-dispatch against a tier
premium applied to every turn of every task. That favors the step-down
tier unless the step-down orchestrator turns out to be procedurally
sloppy — which is the thing not yet measured. The cost side of this
decision comes from data; the quality side comes from design intent.
The tier log's "miss reason" column is where the quality side gets
tested.

### The allowance argument

The top tier's allowance is the scarcest budget in the workflow, and
the person asked that anything the top tier handles be able to fall
back to the step-down tier when it runs out. With the orchestrator one
tier down, only the spec conversation, the planner, and the sign-off
ever draw on that allowance — all short. The fallback is then a
one-line change (drop the override on two dispatches), not a
mid-spec model switch in a long-running session.

### Why medium rather than high

Effort mainly changes how much the model reasons and explores before
acting. For a dispatch loop that is supposed to be hands-off, high
effort pulls the wrong way: it tends to read files itself and
deliberate over things it should delegate, and everything it reads
inflates the context that every later turn re-sends. Medium is the
"do the procedure, don't investigate" setting. The subagent
definitions carry `effort: high`, so implementation, review, and
planning still reason at full strength inside their own short
contexts.

This is the least evidence-backed piece of the policy. Output tokens
were a few percent of the total, so the direct saving from medium is
small; the argument is behavioral, and plausible rather than measured.

### What would change the decision, and in what order

1. **A tier log showing procedural misses by the orchestrator** —
   bundles that missed a file the implementer needed, a review skipped,
   a stale `tasks.md` edit, an escape hatch taken on a task that was
   actually well-specified. First fix: the step-down tier at **high**
   effort. One line in the project's `.claude/settings.json`
   (`effortLevel`), and in `assets/settings-template.json` if it
   should become the default.
2. **Misses that persist at high effort** — then the top tier as
   orchestrator, for one spec of similar size, with the `ccusage
   session --breakdown` comparison afterward. Running the two
   experiments in that order says whether the problem was effort or
   tier, instead of guessing at both.
3. **A policy change that hands the orchestrator judgment calls
   again** — for example, if triage were ever widened so the
   orchestrator resolved design questions inline rather than sending
   them to Plan Mode and the reviewer. That would restore the "fewer
   mistakes" argument and the tier should follow.

Absent one of those, the choice stands.

### Status

Decided September 2026, after two measured specs. Cost side measured;
quality side untested. The next spec's tier log is the first evidence
either way.

---

## Tiering by role at execution time, not by a table written in advance

The reference project's first attempt at model tiering assigned a model
and effort per task, predicted up front from the foundational-vs-
mechanical split in `tasks.md`. It was abandoned: several tasks
assumed safely mechanical benefited from the top tier in ways nobody
saw coming, and a static table had no way to notice. The failure mode
was a lighter model quietly doing a worse job on a task that looked
mechanical — and the table gave that failure zero catches.

The current policy tiers by *role*, decided per task at dispatch time,
and gives the same failure three independent catches: the orchestrator
reads every task before dispatching and every report that comes back
(Step 1 triage); the implementer is under a standing rule to stop and
return the moment it hits a judgment call rather than resolve it; and
the reviewer sits at every phase boundary and on every marked task.
The escape hatch — the orchestrator does the task itself after two
failed verifications or a spurious "judgment call" return, and logs
the miss — is the surviving form of the original lesson: a tier
assignment is a guess to verify, and the misses are the data.

## The first measurement went the wrong way

The first specs measured under the tiering policy cost *more* than the
single-session regime that preceded it, not less. Each cause became a
rule in `SKILL.md`'s loop:

| What happened | The rule it produced |
|---|---|
| An unbounded fix-and-re-review loop in which a cold reviewer found a new objection every round; over half of one spec's subagent tokens sat in four tasks' loops (one task took five rounds) | One review and at most one re-review per invocation; blocking defined narrowly; the re-review sees only the findings and the fix diff |
| Implementers and the orchestrator ingesting raw build and test logs | The constitution names one filtered verification command, run verbatim |
| The orchestrator re-running every verification and reading every diff at the top tier | Verify by running the command, not by reading; open the diff only on failure |
| A whole-codebase pre-merge sweep at the top tier | The sweep is bounded to documents plus the spec's diff, one tier down |
| One orchestrator session accumulating the entire spec | Per-phase `/clear`; spec conversations in their own session |
| "Folded in by the orchestrator" and "seen by the orchestrator" recurring as tier-log entries — the orchestrating session doing work the tiering exists to move off it | The orchestrator does not implement second-look notes or do visual verification by hand |
| The reviewer, at the top tier and reading the codebase fresh, costing about twice the implementation it reviewed | Reviewer defaults one tier down; every review scoped to a bundle |

The rollback condition written into `SKILL.md` follows from this: the
structural argument for dispatch — cheaper rates on the bulk of the
work, small fresh contexts — holds only while the coordination
overhead stays smaller than what it replaces. If a spec measured under
all of the rules above still loses to the single-session regime on the
top tier's budget, the implementer layer goes and the reviewer changes
stay.

## Review cadence: why per-phase everywhere

The cadence used to be per-task throughout foundational phases — a
rule from when the person's attention was the scarce resource and a
per-task look at foundational work cost nothing else. With a
token-priced reviewer it did cost: on the first measured specs, review
ran about twice the implementation it reviewed, and a foundational
phase is short by construction, so its per-phase review comes a day
later rather than the same afternoon, with the pre-merge sweep still
behind it. Per-task review became the exception the planner marks
(`review: per-task`) for a genuinely expensive-to-unwind contract, not
a phase-wide rule.

## Involvement level: why product owner is the default

The skill's original shape had the person approving `plan.md` and
`tasks.md` and pausing after every foundational task — what is now the
technical-lead level. Across the projects the skill has run on, a
person who set out not to touch code turned out not to want to approve
architecture either: the per-task pauses became a report glanced at
and a button pressed, which is worse than no gate because it looks
like review without being one. Product owner became the default, with
the reviewer taking the technical gates and the person keeping the
spec, the attestation at phase pauses, and the escalation triggers.

## Plan and tasks authorship: why it moved to Claude Code

Originally chat authored `plan.md` and `tasks.md` for every spec, in
the same conversation that wrote the spec. That is right for a first
spec, whose plan invents an architecture with nothing to inspect. Once
shipped code is what a plan extends, chat can only see it through a
manually uploaded snapshot that starts subtly stale and drifts from
there; the workaround in practice was a relay — Claude Code prints
state into chat, chat authors the plan against the paste — a
transcription layer whose only function was preserving a rule written
for a condition that no longer held. Drafting moved to the
`sdd-planner`, dispatched once per spec on a planning bundle at the
top tier, so the code exploration a plan needs happens in a
discardable context rather than in the session that carries it into
implementation. Two risks were weighed and judged covered: a planner
with the code open anchoring on what is easy to build (the
skeptical-reviewer's sign-off and the spec-as-contract catch it), and
a product decision being settled silently in `plan.md` (the planner's
standing rule sends it back to the person). `spec.md` stayed a
conversation with the person in both phases — moved to a dedicated
Claude Code session, at the top tier, once chat's reason to exist
(no codebase) was gone.
