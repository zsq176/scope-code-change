# Domain pack example — trading / on-chain transactions

A domain pack extends the main design principles with entries for a specific domain, loaded on demand so other projects never pay for it. This one covers CEX/DEX trading and on-chain transaction systems; to support another domain, derive the analogous pack the same way. Read it in addition to the main design principles.

## New venue types extend the platform, not replace it

- Support for a new venue type (e.g., DEX) extends the existing trading platform — it is not an isolated demo. Reuse the platform's authentication, catalog, parameters/templates, task lifecycle, assets, history, observability, alerts, and recovery.
- Map venues by user capability and lifecycle, not false internal equivalence: a swap is not an IOC order; a transaction hash is not a fill; an AMM pool is not an order book; LP is not a resting order.
- Keep interaction and visual language coherent while representing venue-specific facts honestly — never fabricate order-book, order, or fill semantics for something that has none.
- Support future chains, venues, and protocols through narrow adapters; protocol-specific math stays out of the common execution path.
- Preserve a viable path for the platform's core flows (swap, liquidity provision, volume experiments, cross-market price following).

## Concurrency

- Nonce order does not imply receipt order; verify the user-approved concurrency contract instead of imposing obsolete serialization where independent work is expected.
- Own concurrency at the smallest real resource boundary — nonce, pool state, position, task, and transaction chain are different things.

## Performance evidence

- Latency, availability, reliability, and throughput are trading functionality, not polish.
- Prove trading optimizations with relevant real testnet A/B measurements when authorized; replay and intuition alone do not establish improvement.
- Measure separately where applicable: local computation, locks, queueing, RPC/network, venue acceptance, broadcast, inclusion, confirmation, state projection — separating local delay from external RPC, venue, or chain delay.

## Lifecycle & operations

- Prepared → broadcast → pending → confirmed/failed/unknown, plus stop, restart recovery, and final cleanup, must be observable where relevant.
- Errors identify the relevant wallet/account, asset, operation, venue/protocol response, and termination cause.
- Authorized testnet experiments have a question, measured observations, and a conclusion — don't avoid them merely because they transact, and don't run them aimlessly.
