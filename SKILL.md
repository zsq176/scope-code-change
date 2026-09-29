---
name: scope-code-change
description: Execute a code-change task through strict staged feedback: whole-system impact matrix, line-specific plan, experience gate, evidence-based effectiveness gate, authorized implementation and code audit, then cleanup gate. Do not advance or claim completion without passing each applicable stage.
---

# Scope Code Change

## This is a staged execution instruction

This skill is neither a one-pass checklist nor a general design manifesto. Work on **one stage at a time**. First produce that stage's complete artifact; then audit the artifact; only then enter the next stage. Do not announce that several later gates passed in one final answer. Prior experience constrains a proposed plan, but does not choose the plan in advance.

At the end of **every** stage, visibly report in commentary:

- **Result:** the complete artifact or a link to it, with the decisive findings.
- **Exit:** the condition required to leave this stage.
- **Audit:** PASS or FAIL, with evidence and any unresolved item.
- **Next / return:** the exact stage being entered, or the stage to which work returns.

A FAIL does not become PASS by rewording the verdict. Re-enter only after changing the artifact, evidence, or hypothesis. If a required experiment is impossible within the user's authorization, report that blocker; do not loop over the same unproven proposal or claim completion.

A request for a plan does not authorize production code edits, deployment, or live trading. Diagnostic tests and experiments must remain within the user's authorization. If evidence requires additional authority, ask before taking that action.

## Stage 1 — whole-system impact matrix

**Do this first, before choosing an implementation.** Inspect the actual repository, current behavior, user workflow, relevant runtime evidence, and worktree. Identify the project's primary capability and what the request must change or preserve. The impact matrix is a forward-looking constraint: it prevents a local solution from silently changing the rest of the system.

**Result:** publish a matrix for every discovered subsystem, including materially unaffected ones. Each row states current owner and code evidence, whether it may change, affected inputs/outputs and lifecycle transitions, invariants at risk, and the adaptation or regression check needed. Cover, where present: ingestion and market data; strategy/signals; risk/admission; order or transaction creation/amendment/cancellation/execution; fills and partial fills; positions/exposure; accounting/PnL; persistence/projections; API/frontend; configuration/reload; logs/metrics; startup/reconnect/recovery/shutdown; concurrency/resource ownership; runtime modes; tests/deployment; and operator workflow. Group unaffected areas only when the same evidence and reason genuinely apply.

End with the **impact conclusion**: the exact system boundaries the future plan must respect, known failure or capability gap, credible unknowns, and where evidence is still needed. Do not smuggle the preferred fix into this conclusion.

**Exit audit:** verify the traced producer → state → consumer paths and applicable lifecycle are complete; affected and unaffected areas have reasons; no core behavior or second-order effect is missing. On FAIL, remain in Stage 1 and complete the matrix. On PASS, enter Stage 2.

## Stage 2 — concrete, indivisible plan

Build the plan **under the Stage 1 impact conclusion**, not from abstract principles. An atomic goal is the smallest independently verifiable behavior change; it may touch multiple files, but must not combine separable outcomes. Split a goal joined by "and" unless the two changes cannot work or be verified separately.

**Result:** output the complete plan. For every atomic goal provide:

1. Current evidence and the precise cause or missing transition; separate fact, inference, and unknown. For a bug: trigger → actual path → first invalid transition → symptom. If cause is unproven, plan a discriminating experiment instead of a speculative repair.
2. Exact current code target as **file:line plus symbol/function**. State the lines or operation to remove, replace, or insert and the resulting behavior/data/state transition. For a new file, name its path, owner, calling site, and insertion point. Line numbers refer to the inspected baseline and must be rechecked before editing.
3. Required interfaces, conditions, normal path, failure path, stop/restart/recovery behavior, and the old logic to remove or preserve where applicable.
4. Observable acceptance for that one goal, including the original user scenario and any necessary performance/concurrency measure.
5. Its dependency on other goals, material code/concept cost, and explicit non-changes.

The plan must not contain standalone "principles," generic architecture theory, or untestable directions such as "optimize," "strengthen," "reuse," or "add retries." Put principles in Stage 3; put concrete modifications here. A reader must know what will change, at which code sites, and how to verify it.

**Exit audit:** every goal is atomic, line-specific, consistent with the Stage 1 matrix, and has an observable result. On local incompleteness, revise Stage 2 and output the full changed plan. If a new effect or owner is discovered, return to Stage 1. On PASS, enter Stage 3.

## Stage 3 — experience gate, then revised plan

Now apply the experience registry below **to the plan that Stage 2 already produced**. This gate is for high-frequency failure patterns and developer learning, not for mechanically copying an old solution.

**Result:** publish an experience matrix with applicable principle IDs, the plan decision being tested, repository/runtime evidence, PASS/FAIL/UNKNOWN, and the exact correction or test needed. Explicitly check whether the plan preserves the primary capability, first-usable behavior, required concurrency/latency, real ownership, code economy, and complete phase delivery. Do not include a principle merely to fill a row, and do not omit an applicable one.

If the matrix exposes a weakness, change the plan and **output the full revised plan**, not just a list of comments. Re-audit the changed rows. If the revision changes system impact, return to Stage 1 and regenerate the downstream plan. UNKNOWN on a design-critical principle is not PASS.

**Exit audit:** all applicable design constraints are supported; the revised plan still consists of precise atomic goals. On FAIL, loop through Stage 2 → Stage 3, or Stage 1 if system effects changed. On PASS, enter Stage 4.

## Stage 4 — effectiveness gate

Before treating a plan as valid, prove the **key proposed mechanism**, not merely its plausibility. The evidence must be non-inferential: a reproduction with a discriminating test, measured data, a controlled comparison/A/B experiment, or a directly observed equivalent path under matching conditions. Code reading and documentation can explain a mechanism, but cannot alone prove that a proposed runtime or performance fix works. For trading-performance claims, use relevant real testnet measurements when authorized and separate local delay from external RPC, venue, or chain delay.

A small reversible experiment may test the mechanism; it is not the delivered implementation and its artifacts must later be cleaned. Record baseline, changed condition, observed result, sample/limits, and why the result supports each critical plan goal. Do not substitute one happy-path run for lifecycle or stress evidence when those are the claim.

**Result:** output the evidence and a per-critical-goal effectiveness verdict. State exactly what is proven, what is not, and whether the proposed root fix is supported.

**Exit audit:** PASS only when non-inferential evidence supports every critical mechanism. If evidence contradicts the plan, is absent, or cannot be obtained, mark FAIL and return to **Stage 1**, revising the impact understanding and hypothesis before producing another plan. Do not cycle through the same plan without a new discriminating step. If no authorized route to evidence exists, report the blocker rather than fabricate a PASS.

On PASS:
- If the user requested **only a plan**, skip the code implementation and code audit stages and enter Stage 7.
- If the user authorized a **code change**, enter Stage 5.

## Stage 5 — implement the authorized plan

Inspect and preserve unrelated worktree changes. Recheck Stage 2 line references, then implement the approved atomic goals in their durable owners. Do not silently change semantics, add broad serialization, or expand scope outside the approved plan. Remove replaced behavior instead of maintaining parallel paths.

For a multi-goal plan, complete and verify one atomic goal before beginning the next. Report its actual diff and immediate evidence using Result → Exit → Audit → Next/return. If implementation reveals a false plan assumption or new system impact, return to Stage 1 rather than disguising a redesign as a small code adjustment.

After all goals are implemented, enter Stage 6. Implementation alone is not acceptance.

## Stage 6 — code audit gate

**Result:** map every Stage 2 goal to actual changed file:line locations and demonstrate that its intended behavior and acceptance criteria were implemented. Review the diff for bugs, concurrency/resource hazards, error propagation, lifecycle and recovery, permissions/data integrity, performance, and regressions in the relevant unchanged path. Run the promised tests and reproduce the Stage 4 effectiveness result in the implemented path.

**Exit audit:** PASS only if the code follows the approved plan, has no material defect found in review, passes the agreed acceptance tests, and reproduces the effectiveness evidence. A build alone is insufficient. On implementation FAIL, return to Stage 5, fix and repeat Stage 6. If the evidence invalidates the plan itself, return to Stage 1. On PASS, enter Stage 7.

## Stage 7 — cleanup gate

This gate applies to **both** plan-only and implementation tasks. For plan-only work, check that diagnostic experiments left no artifacts and the final plan has no redundant design or unsupported scope. For implemented work, inspect intermediate scripts, temporary test files, process artifacts, obsolete branches/adapters/states/configuration, duplicated logic, overdesign, dead code, and code that can be simplified without changing behavior. Follow the project's rule for whether permanent tests belong in the repository.

**Result:** list what was removed or simplified and what remains intentionally. Verify that the final artifact is the smallest complete expression of the proven plan.

**Exit audit:** on FAIL, re-enter Stage 7 and clean again. If cleanup changes behavior, rerun Stage 6 and any affected effectiveness checks before returning here. On PASS, end the workflow and hand off the result. The final answer must report the passed path, evidence, and any honest external limit; it cannot replace the staged feedback already given.

## Experience registry for Stage 3

These are the accumulated design and development lessons. They are constraints on a proposed plan, not a script for choosing it. Apply the CEX/DEX-specific entries only in market-value-management; for another project derive the analogous project-specific lessons instead.

### Product and delivery

1. **First usable, then pleasant to use.** Usability includes intended behavior, required concurrency, acceptable latency, stop/restart/recovery, and understandable results. "No error" is not enough.
2. Do not deliver a demo or disposable "minimum closed loop" as a finished phase. A small phase must be complete to its approved test-environment standard.
3. Complete, test, accept, and clean one phase before extending it; deliberate work beats repeated broad rework.
4. Each phase must advance the intended whole system, not leave temporary owners, duplicate paths, a hard-coded market, unused placeholders, or deferred cleanup.
5. Do not silence failure by removing core function, hiding work, waiting without bound, or silently skipping actions.

### CEX/DEX project trunk — market-value-management only

6. DEX extends the existing CEX trading system; it is not an isolated AMM demo. Reuse matching authentication, catalog, parameters/templates, task lifecycle, assets, history, observability, alerts, and recovery capabilities.
7. Map CEX and DEX by user capability and lifecycle, not false internal equivalence. Swap is not an IOC order; a transaction hash is not a fill; an AMM pool is not an order book; LP is not a resting order.
8. Support future chains, venues, protocols, and markets through narrow adapters; protocol-specific math does not belong in the common execution path.
9. Preserve a viable path for Swap, liquidity provision, volume experiments, and cross-market price following.

### Function and concurrency

10. The feature must work before its protection is called successful; a safeguard that disables it is not a repair.
11. Verify the current user-approved concurrency contract. Where independent work is required, nonce order alone does not imply receipt-by-receipt serialization; do not impose obsolete concurrency requirements.
12. Serialize only real dependencies, such as one position mutation or an ID/parameter produced by a prior transaction; a shared pool alone is not dependency.
13. Own concurrency at the smallest real resource boundary: nonce, pool state, position, task, and transaction chain differ.
14. Stop cancels unbroadcast work promptly; broadcast work remains tracked to a definite outcome.

### Performance

15. Latency, availability, reliability, and throughput are trading functionality, not later polish.
16. Reject unbounded queues, cooling, sleeps, broad serialization, and silent throttling as defect camouflage; necessary queues need bounds, measured delay, and saturation behavior.
17. Prove trading optimizations with relevant real testnet A/B evidence when authorized; replay and intuition alone do not establish improvement.
18. Measure local computation, locks, queueing, RPC/network, exchange acceptance, broadcast, inclusion, confirmation, and state projection separately where applicable.

### Root-cause repair

19. Identify original input, lifecycle, first invalid transition, false assumption, and the smallest transition that removes the cause.
20. Eliminate the generator before adding retries, cooling, guards, compensation, broad locks, or extra states; containment is not root repair.
21. Check analogous protocols, algorithms, venues, and shared paths for the same cause without expanding edits beyond evidence.
22. After failed fixes, stop patching; return to raw input and remove ineffective changes.

### Architecture and code economy

23. Preserve owner boundaries among data/RPC/WS, protocol, execution/recovery, strategy, API, persistence, and presentation.
24. Abstract stable shared capability and a narrow adapter, not one algorithm, protocol, page, or incident.
25. Directory and dependency structure are deliverables; no provisional directory, catch-all file, leftover utility/test script, or "later" cleanup.
26. Budget files, lines, states, and concepts against outcome value; a local bug does not automatically justify a large state machine.
27. Delete replaced branches, locks, adapters, duplicate state, and temporary validation artifacts.

### Interface and operations

28. Start from the operator's action and question; for UX, inspect the existing surface before inventing another page or backend subsystem.
29. Keep CEX/DEX interaction and visual language coherent while representing AMM-specific facts honestly; do not fabricate order-book, order, or fill semantics.
30. Errors identify relevant wallet/account, asset, operation, venue/protocol response, and termination cause; "check logs" is not an operator answer.
31. Prepared, broadcast, pending, confirmed, failed, unknown, stop, restart recovery, and final cleanup must be observable where relevant.

### Testing and acceptance

32. Test the user's original scenario, not just compilation, isolated tests, or one successful transaction.
33. Acceptance covers required concurrency, normal/failure/stop/restart/recovery and final state or asset reconciliation where relevant.
34. Authorized real testnet experiments have a question, measured observations, and conclusion; do not avoid them merely because they transact or run them aimlessly.
35. Complete only after the applicable Stage 1–7 exit audits pass. Evidence, not confidence or a one-shot summary, determines each verdict.
