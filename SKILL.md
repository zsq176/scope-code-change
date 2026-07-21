---
name: scope-code-change
description: Scope and right-size every code change before implementation. Use before writing or modifying code for features, bug fixes, refactors, optimizations, or behavior-changing configuration so the change respects the project's primary purpose, requirement priority, whole-system impact, cross-layer adaptation, complexity budget, and long-term maintainability.
---

# Scope Code Change

Before editing code, establish:

- Project trunk: the primary behavior the project exists to deliver.
- User outcome: restate what must change in plain language, including what the user is not asking for.
- Requirement rank: trunk, supporting, secondary, or rare tail event.
- Root cause: identify the first transition where valid state becomes invalid and the incorrect assumption responsible.
- Minimum outcome: the smallest end-to-end behavior that satisfies the request.
- Change budget: expected files, approximate added lines, and new concepts.

State these briefly in commentary before implementation. Base them on the current repository, not the request in isolation.

## Whole-system impact gate

Treat every code change as a system change. Before editing:

- Map the affected producers, consumers, state, persistence, APIs, configuration, logs and metrics, startup/reload/shutdown paths, runtime modes, and operator or UI behavior.
- Identify the shared invariants and every required adaptation. Do not update one representation while leaving another stale.
- Check second-order effects on correctness, performance, reliability, security, data consistency, operations, and maintenance.
- Prefer a globally coherent change over a locally elegant one. Reject a local optimization that shifts complexity, ambiguity, or failure into the rest of the project.

Keep the assessment proportional to the change, but do not start implementation until the local change's role in the whole system is clear.

## Elimination-first

Before designing how to handle a problem, identify the mechanism that repeatedly creates it and ask whether that mechanism can be removed or made impossible. Prefer restoring the violated invariant, removing the bad input or assumption, or deleting the unnecessary state/path over adding guards, retries, repair jobs, and recovery branches downstream.

Treat recurring instances of the same symptom as evidence that the generating mechanism still exists. Better containment is not progress toward elimination. Add containment or recovery only after the source is removed, or after concrete evidence shows that the source is external and cannot be eliminated within scope.

## Root-cause gate

For bug fixes, do not edit behavior until all of these can be stated concretely:

1. The original failure and the exact path that produces it.
2. The first point where correct input becomes incorrect state or output.
3. The wrong code assumption, supported by logs, a reproduction, or observed values.
4. The smallest state transition that removes the cause.

Classify every proposed change as:

- **Root fix:** prevents the invalid state from being created.
- **Containment:** limits damage after the state is already invalid.
- **Recovery:** restores state after failure.
- **Evidence:** makes an unknown link observable.

Containment, recovery, and evidence may be useful, but must not be presented or accumulated as the root fix. If evidence is insufficient, add only the minimum observability needed and wait for proof before changing behavior.

After each attempted fix, test the user's original symptom. If it remains, invalidate or revert the attempt before adding another. After two failed attempts, stop patching and rebuild the causal chain from the raw input boundary. A fix is not understood if it cannot be explained in one sentence without words such as "somehow", "probably", or "should".

## Decision rules

1. Protect the trunk. A secondary feature must not materially enlarge, slow, obscure, or destabilize the primary system.
2. Eliminate the generator first. Prefer making the failure impossible over detecting, repairing, retrying, or limiting it forever.
3. Reuse existing paths. Do not add a new layer, subsystem, ledger, state machine, or framework unless the minimum outcome cannot work without it.
4. Implement the common path first. Handle failures that are likely or costly; defer low-probability tail events until they occur or the user explicitly prioritizes them.
5. Keep secondary features deliberately modest. Functional and maintainable is enough; do not perfect them at the expense of the main program.
6. Preserve semantics outside the request. Do not redesign strategy, architecture, or execution behavior merely to make the local change cleaner.
7. Match verification to risk. Test the changed path and critical invariants; do not expand into an unrelated review.
8. Fix the producer before the consumer. When bad state is detected downstream, first correct the code that creates it; do not grow downstream guards unless the root fix cannot cover a costly failure mode.

## Complexity guard

Compare the proposed change with the whole repository. If a supporting or secondary feature approaches a material fraction of the project, introduces several new concepts, or exceeds roughly twice the initial line budget, stop editing and simplify the design.

For a localized bug, broad recovery machinery, new state machines, or several hundred lines are presumptively out of scope. Require concrete evidence that the root cause spans that much surface before proceeding.

Prefer fewer states, fewer branches, fewer files, and one obvious data flow. Treat maintainability as part of the requirement, not cleanup after it.

## Post-change system review

Before delivery:

1. Review syntax and build validity, code quality, obvious bugs, lifecycle and resource cleanup, concurrency hazards, and dead or superseded paths.
2. Test the changed behavior against the original whole-system assessment and user outcome, including affected integrations and runtime modes. Delete temporary test files and scaffolding immediately after use.
3. Recheck every producer, consumer, state, configuration, log, metric, documentation, and operational adaptation identified before editing.
4. Re-evaluate the root cause and the whole-system result. Confirm that the change removes the generating mechanism or restores the invariant, and that the changed local part now serves the system better.
5. Reject or simplify the change if a local metric improved while overall correctness, performance, reliability, clarity, or maintainability became worse.
6. Confirm the main path remains understandable, the diff stayed near its budget, and no temporary artifacts remain.
