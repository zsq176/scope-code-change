---
name: scope-code-change
description: Scope and right-size every code change before implementation. Use before writing or modifying code for features, bug fixes, refactors, optimizations, or behavior-changing configuration so the change respects the project's primary purpose, requirement priority, complexity budget, and long-term maintainability.
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

## Before delivery

Check that:

- The requested behavior works end to end.
- The main path remains understandable and intact.
- The diff stayed near its stated budget.
- No temporary tests, debug files, or scaffolding remain.
