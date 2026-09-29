---
name: scope-code-change
description: Execute a code-change task through a strict staged process — impact analysis, atomic line-specific plan, evidence-gated implementation, code audit, cleanup — with depth scaled to task size, grounded in a reusable set of development principles. Never advance past a failed check or claim completion without evidence.
---

# Scope Code Change

This skill binds two things together, and both are mandatory:

1. **A strict staged process.** Work one stage at a time: produce that stage's complete artifact, audit it, and only then advance to the next stage. A FAIL stays FAIL until the artifact, the evidence, or the hypothesis has actually changed — a verdict is never reworded into a pass. If the evidence you need is outside your authorization, report that blocker instead of looping on an unproven proposal or claiming completion.
2. **A reusable development philosophy.** The principles at the end of this file are accumulated lessons from past failures. They constrain every plan; they do not choose a solution in advance. Check plans against them; do not recite them.

Scale the depth to the task. Small tasks take the fast track and skip the paperwork. The evidence and honesty requirements never shrink.

## Design philosophy

Six rules from which everything else follows:

1. **The system comes first.** Understand what a change touches — including what it must not disturb — before choosing an implementation. A local fix that silently reshapes another subsystem is the most expensive kind of failure.
2. **Plans are concrete and atomic.** Each unit of work is the smallest independently verifiable behavior change, pinned to exact code sites, with observable acceptance criteria. Vague directions ("optimize", "harden") are not plan content.
3. **Evidence decides.** "It works / it's fixed / it's faster" is established by observation — a passing scenario, a measured comparison, a reproduced bug — never by a plausible argument alone.
4. **Remove causes, not symptoms.** Containment (retries, guards, cooldowns, broad locks) hides a failure; it does not repair it. Find the first invalid transition and remove its generator.
5. **Be economical.** The deliverable is the smallest complete expression of the proven plan. Replaced code, temporary scaffolding, and unused generality are deleted, not left behind.
6. **Be honest.** Report what the evidence shows, including its limits. Completion is claimed only along a path of passed checks — never by a final summary alone.

## Task sizing

Classify the task before starting:

- **Fast track** — the change is small and fully predictable: one or two known code sites, behavior known in advance, verification is a single run. Examples: a typo, a log line, a config value, an isolated bug with an obvious cause.
- **Full pipeline** — everything else: multiple files, non-obvious failure modes, cross-subsystem impact, concurrency or performance claims, or any task the user marks as major.

When in doubt, use the full pipeline. A request for a plan only never authorizes production edits, deployment, or transactions; diagnostics stay within the user's authorization — if the evidence needs more authority, ask.

## Fast track

Four steps, no gate reports:

1. **Impact check** — search callers, shared state, and configuration for hidden coupling. This is a search, not a matrix.
2. **Change** — make the edit at the known site.
3. **Verify** — run the scenario or test that proves the change works.
4. **Report** — state what changed, what the evidence is, and what was cleaned up.

If the impact check uncovers more coupling than expected, or verification contradicts the intent, switch to the full pipeline.

## Full pipeline

Each stage ends with a one-line gate report before the next stage begins:

> `Stage N: PASS` — decisive evidence · next stage  (or)  `Stage N: FAIL` — what is missing · which stage to re-enter

**Stage 1 — Impact analysis.** Build a matrix with one row per subsystem — including subsystems expected to be unaffected: owner and code evidence, whether it may change, invariants at risk, and the regression check needed. Cover, where present: data ingestion, domain logic, validation, request/transaction lifecycle, derived state, accounting, persistence, API/frontend, configuration, observability, startup/recovery/shutdown, concurrency ownership, and operator workflow. End with the exact system boundaries the plan must respect. Do not let a preferred solution shape this analysis.

**Stage 2 — Atomic plan.** For every goal: the cause or missing transition (separate fact, inference, and unknown), exact targets as `file:line` + symbol, the resulting behavior and state transitions on normal/failure/recovery paths, acceptance criteria including the user's original scenario, dependencies on other goals, and explicit non-changes. If the cause is unproven, plan a discriminating experiment instead of a speculative repair. Exit check: every goal is atomic and line-specific, and the plan holds against the design principles — violations are fixed by revising the plan here.

**Stage 3 — Effectiveness evidence.** Prove each critical mechanism with non-inferential evidence: a discriminating reproduction, a measured A/B comparison, or direct observation under matching conditions. Record baseline, changed condition, result, and limits. Reading code can explain a mechanism; it cannot prove a runtime or performance claim. A small reversible experiment may serve as evidence; its artifacts are temporary and get cleaned up. If no authorized route to the evidence exists, report the blocker. Plan-only tasks end here.

**Stage 4 — Implementation.** Recheck the line targets, then implement one goal at a time, verifying each before starting the next. Do not expand scope beyond the approved plan. If implementation disproves a plan assumption, return to Stage 1 or Stage 2 — a redesign disguised as a small adjustment is itself a defect.

**Stage 5 — Audit.** Map every goal to the actual changed lines. Review the diff for defects, regressions, and concurrency hazards. Run the acceptance criteria and reproduce the Stage 3 evidence on the delivered path. A passing build is not acceptance.

**Stage 6 — Cleanup.** Remove experiment artifacts, replaced branches, duplicate state, and overdesign. Confirm the result is the smallest complete expression of the proven plan. If cleanup changed behavior, re-run Stage 5. The final answer then reports the passed path, the evidence, and any honest limits — it supplements the gate reports; it does not replace them.

**Return discipline.** A failed stage sends work back to the stage that owns the broken assumption — normally Stage 1 (system understanding) or Stage 2 (the plan). Never patch forward from a broken plan, and never repeat a failed proposal without a new discriminating step.

## Design principles

The reusable lessons. Each one constrains plans and reviews; none chooses a solution in advance. Apply the ones that are relevant; do not pad.

### Delivery

1. First usable, then pleasant: intended behavior, required concurrency, acceptable latency, understandable results — including stop/restart/recovery. "No error" is not done.
2. A phase is delivered complete to its agreed standard, or not delivered. No demo as product, no temporary owners, no deferred cleanup.
3. Never mask failure by removing core function, hiding work, waiting without bound, or silently skipping steps.

### Root cause

4. Trace: original input → lifecycle → first invalid transition → the smallest change that removes the cause.
5. Containment is not repair. Remove the generator before adding retries, guards, compensation, or broad locks.
6. After a failed fix, stop patching: return to the raw input, back out the ineffective changes, and re-derive.
7. Check analogous protocols and shared paths for the same cause — without expanding edits beyond the evidence.

### Concurrency

8. Serialize only true dependencies: a single writer of a resource, or a value produced by a required earlier step. Sharing a resource pool alone is not a dependency, and submission order does not imply processing order.
9. Own each lock at the smallest real resource boundary. Different resources — sequence tokens, shared state, tasks, request chains — are different owners.
10. Cancellation is prompt for work not yet externally visible; work already visible is tracked to a definite outcome and never abandoned.

### Performance

11. Latency, availability, and throughput are functionality, not later polish.
12. Unbounded queues, sleeps, and silent throttling are defect camouflage. A necessary queue has bounds, measured delay, and defined saturation behavior.
13. An optimization claim requires a measured before/after, with delay attributed to the correct layer: local computation, lock, queue, network, or external service.

### Architecture and economy

14. Preserve ownership boundaries: data access, protocol/integration, domain execution, application logic, API, persistence, presentation.
15. Abstract stable, shared capability behind narrow interfaces — not one algorithm, one page, or one incident.
16. Budget files, lines, states, and concepts against the value they produce. A local bug does not justify a state machine.
17. Directory and dependency structure is part of the deliverable. No catch-all files, leftover scripts, or "later" cleanup.

### Errors and operations

18. An error message names the object, the operation, the upstream response, and the termination cause. "Check the logs" is not an operator answer.
19. Lifecycle states — starting, normal, degraded, stopping, recovering, cleaning up — are observable where they matter.

### Testing

20. Acceptance tests the user's original scenario and its lifecycle: normal, failure, restart/recovery, and final-state reconciliation. Compilation and one happy path prove nothing.
