---
name: scope-code-change
description: Execute code-change tasks with rigor proportionate to risk — whole-system impact awareness, line-specific plans, evidence-based verification, honest reporting. For non-trivial changes (multi-file behavior, concurrency/performance claims, cross-subsystem impact); trivial edits take a lightweight path with no ceremony. Never claims completion without evidence.
---

# Scope Code Change

## Core philosophy

Five ideas govern every change, at every size:

1. **Know the system before editing it.** A change lives inside a larger system; find what else it touches — including what it must *not* disturb — before choosing an implementation.
2. **Evidence over confidence.** "It works / it's fixed / it's faster" requires an observation: a passing scenario, a measured comparison, a reproduced bug. A plausible argument is not evidence.
3. **Repair the cause, not the symptom.** Remove the generator of a failure before reaching for retries, guards, cooling, or broad locks. Containment is not repair.
4. **Work in verifiable atoms.** Each unit of change is the smallest independently verifiable behavior change, with a known code site and an observable acceptance check.
5. **Be economical and honest.** Deliver the smallest complete expression; delete what is replaced. A failed check stays failed until the artifact actually changes — never reword a verdict into a pass, and report limits instead of claiming completion.

Rigor scales with blast radius, not with habit. Pick the lightest path below that covers the task's risk: skipping ceremony on a small task is correct; skipping evidence never is.

## Choose a path

Spend a minute gauging blast radius, then commit:

- **Light path** — small and well-understood: one function or file, obvious behavior, easy verification (typo, log line, copy, config value, isolated bug).
- **Standard path** — a behavioral change across a few files, or anything whose failure modes aren't obvious at a glance. This is the default.
- **Full path** — architecture or concurrency changes, performance claims, changes crossing subsystem boundaries, or work the user marks as major.

When torn between two paths: take the heavier one for analysis, the lighter one for ceremony. A request for a plan only never authorizes production edits, deployment, or transactions; diagnostic work stays within the user's authorization — if evidence needs more, ask.

## Light path

1. Glance at callers and related state for hidden coupling — a search, not a matrix.
2. Make the change; verify by running the affected scenario or test.
3. Report what changed and the evidence it works. Done.

## Standard path

Four questions, in order. Answer each in a few sentences before acting on the next. When an answer exposes a problem, fix that artifact and re-answer; when it invalidates an earlier one, go back upstream — don't patch a broken plan forward. If the user asked only for a plan, stop after question 2.

1. **Impact** — which subsystems, data flows, and behaviors does this touch, including ones it must not disturb?
2. **Plan** — per atomic goal: the cause (label facts / inferences / unknowns), the exact target as `file:line` + symbol, the resulting behavior, and observable acceptance. If the cause is unproven, plan a discriminating experiment instead of a speculative repair. No untestable directions — "optimize" and "harden" are either concrete goals or they don't belong.
3. **Verify** — prove the key mechanism: reproduce the bug, run the discriminating test, measure the comparison. Code reading alone cannot prove a runtime or performance claim. A small reversible experiment is fine (within authorization; its artifacts get cleaned up later) — it is not the delivery.
4. **Audit & clean** — the diff matches the plan, acceptance passes, replaced code is deleted, no leftover scripts or temp files.

## Full path

For high-risk or system-wide work, the same four questions become published stages. Each stage ends with a one-line gate report before the next begins:

> **Stage N: PASS/FAIL** — decisive evidence · next stage (or the stage being returned to)

A FAIL only clears by changing the artifact, evidence, or hypothesis. If no authorized route to the needed evidence exists, report the blocker — never loop on an unproven proposal or fabricate a pass.

**Stage 1 — Impact matrix.** One row per subsystem, *including materially unaffected ones*: owner + code evidence, may-change or not, invariants at risk, regression check needed. Cover where present: ingestion/data, domain logic, risk/validation, request/transaction lifecycle, derived state and balances, accounting, persistence/projections, API/frontend, config/reload, observability, startup/reconnect/recovery, concurrency ownership, operator workflow. Close with the boundaries the plan must respect — without smuggling in a preferred fix.

**Stage 2 — Atomic plan.** Per goal: evidence and cause (fact/inference/unknown), `file:line` + symbol targets, behavior and state transitions on normal/failure/restart paths, acceptance including the user's original scenario, dependencies, and explicit non-changes.

**Stage 3 — Experience check.** Apply the design principles below to the produced plan. Record only applicable principles, each with the plan decision it tests and any correction. A weakness found → revise and republish the changed goals (unchanged goals stand; say so). System impact changed → return to Stage 1.

**Stage 4 — Effectiveness evidence.** Non-inferential proof of each critical mechanism: discriminating reproduction, measured A/B, or direct observation under matching conditions. Record baseline, changed condition, result, and limits. Plan-only tasks end here, after cleaning up experiment artifacts.

**Stage 5 — Implement.** Recheck line targets, then implement goal by goal, verifying each before the next. Discovering a false plan assumption is a return to Stage 1, not a quiet redesign disguised as a small adjustment.

**Stage 6 — Audit.** Map every goal to the actual changed lines; review the diff for defects, regressions, and concurrency hazards; run acceptance and reproduce the Stage 4 evidence on the delivered path. A passing build is not acceptance.

**Stage 7 — Cleanup.** Remove experiment artifacts, dead branches, duplicate state, and overdesign; confirm the result is the smallest complete expression of the proven plan. If cleanup changed behavior, re-audit. Then hand off: the final answer reports the passed path, evidence, and honest limits — it supplements, not replaces, the gate reports.

## Design principles

A checklist for Standard question 2 and Full Stage 3. Apply what's relevant; don't pad rows to fill a matrix. Domain-specific entries live in optional packs under `references/`, loaded on demand — this repo ships a trading / on-chain transaction example ([references/market-trading.md](references/market-trading.md)); for another domain, derive the analogous pack the same way.

**Product**

- First usable, then pleasant: intended behavior, required concurrency, latency, stop/restart/recovery, understandable results. "No error" is not done.
- A phase ships complete to its approved standard — no demo-as-finished, no temporary owners, duplicate paths, or deferred cleanup.
- Never silence failure by removing core function, hiding work, waiting without bound, or silently skipping steps.

**Concurrency**

- Serialize only real dependencies (a mutation of one shared resource, an identifier produced by a prior step); merely sharing a resource pool does not create a dependency, and submission order does not imply processing order.
- Own concurrency at the smallest real resource boundary — sequence tokens, shared state, tasks, and multi-step request chains are different things.
- Stop cancels unbroadcast work promptly; broadcast work is tracked to a definite outcome.

**Performance**

- Latency, availability, and throughput are functionality, not later polish. Unbounded queues, sleeps, broad serialization, and silent throttling are defect camouflage, not fixes.
- An optimization claim needs a measured before/after, with local delay separated from network or external-service delay.

**Root cause**

- Trace original input → lifecycle → first invalid transition → the smallest change that removes the cause. After a failed fix, stop patching; return to the raw input and back out ineffective changes.
- Check analogous protocols and shared paths for the same cause — without expanding edits beyond the evidence.

**Architecture & economy**

- Preserve owner boundaries (data/RPC, protocol, execution, strategy, API, persistence, presentation); abstract stable shared capability behind a narrow adapter, not one algorithm or incident.
- Budget files, lines, states, and concepts against outcome value — a local bug doesn't justify a state machine.
- Errors name the relevant object, operation, upstream response, and termination cause; "check the logs" is not an operator answer.

**Testing**

- Test the user's original scenario and its lifecycle — normal, failure, restart/recovery, final-state reconciliation — not just compilation or one happy path.
