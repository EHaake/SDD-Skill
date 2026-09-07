# Why the orchestrator runs one tier down, at medium effort

The decision record for the model policy's most-questioned choice.
Nothing here is needed to *apply* the policy — `SKILL.md`'s "Model
tiering" section has the rules. Read this when the choice is being
revisited, or when a tier log makes it look wrong.

## The question

If the top tier is the best model, why isn't it the orchestrator? A
better orchestrator makes fewer mistakes, and fewer mistakes are
cheaper in the long run even if each turn costs more up front. That
argument is sound wherever it applies. The policy is built so that it
applies as little as possible.

## What the measurement showed

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

## Why "fewer mistakes" mostly doesn't apply

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

## The allowance argument

The top tier's allowance is the scarcest budget in the workflow, and
the person asked that anything the top tier handles be able to fall
back to the step-down tier when it runs out. With the orchestrator one
tier down, only the spec conversation, the planner, and the sign-off
ever draw on that allowance — all short. The fallback is then a
one-line change (drop the override on two dispatches), not a
mid-spec model switch in a long-running session.

## Why medium rather than high

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

## What would change the decision, and in what order

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

## Status

Decided September 2026, after two measured specs. Cost side measured;
quality side untested. The next spec's tier log is the first evidence
either way.
