# Skill: Dexie Routing and External Offer Execution

> Repo knowledge for Dexie as an external Chia liquidity venue.
> Use this when working on Forge routing, external offer execution, or price discovery.

---

## Domain

This skill covers the Dexie side of Chia liquidity aggregation:

- exact-size XCH/CAT quoting
- external offer submission
- token registry and asset normalization
- orderbook-aware routing heuristics
- how Dexie differs from Forge pool execution

Use it for:
- `dexieAdapter.ts` or related aggregator work
- external quote and execution UX
- price-monitor and oracle quests that include Dexie
- deciding when to route to Dexie instead of Forge pools

---

## What Dexie Is

Dexie is an offer-driven external liquidity venue. From Forge's point of view, it is not a pool contract that we spend directly. It is an external settlement lane that accepts Chia offers and either matches or routes them.

Practical consequences:

- Dexie execution is offer-first, not pool-spend-first
- the route may expire if price or available liquidity moves
- user-facing UX needs quote freshness and retry handling
- XCH/CAT is the clearest supported route shape to assume first

---

## High-Value References

- https://dexie.space/api/swap
- https://api-testnet.dexie.space/v1
- https://dexie.space/markets
- [build-forge-swap-execution-path.md](../quests/build-forge-swap-execution-path.md)
- [build-forge-swap-aggregator.md](../quests/done/build-forge-swap-aggregator.md)

Keep these links in the skill because they are the fastest way to re-anchor exact quote and submit behavior.

---

## Working Model

### Quote lane

Use Dexie for an exact-size external quote when one side is native XCH and the other is a CAT.

Typical request shape:

```text
GET /swap/quote?from={from_asset}&to={to_asset}&from_amount={amount}
```

Typical response fields to care about:

- `from_amount`
- `to_amount`
- `combination_fee`
- `suggested_tx_fee`

These should be treated as execution-context inputs, not just display metadata.

### Submission lane

Forge creates or imports an offer locally, then submits that offer to Dexie.

Typical request shape:

```text
POST /swap
```

Expected payload concept:

- `offer`
- optional fee destination metadata when supported by the lane

Dexie may reject if the market moved or if the offer no longer matches the quoted route.

---

## Routing Heuristics

Use Dexie when:

- the route is XCH to CAT or CAT to XCH
- the external quote beats Forge-only execution
- the user has an offer-capable execution lane
- the route benefits from exact-size external matching instead of pool math

Prefer Forge when:

- liquidity is already in a known local Forge pool
- the route is a spend-bundle-native pool operation
- the external quote is stale or unavailable
- deterministic local execution matters more than external best price

Important boundary:

- Dexie is an external-offer venue
- Forge pool add/remove/swap is a pool spend-bundle venue
- mixed-source routes need explicit execution handling instead of pretending both lanes are the same

---

## Asset Normalization

Be explicit about native asset normalization.

Rules to keep in mind:

- Dexie commonly refers to native XCH as `xch`
- local testnet and wallet code may use `txch`
- Forge internals may use the zero asset id for native handling in pool state

Any adapter should normalize these forms consistently before quoting or submitting.

---

## UX Patterns Worth Keeping

Dexie routes need:

- quote freshness state
- expiry-aware retry messaging
- surfaced fee context
- explicit venue labeling in route summaries
- graceful fallback to Forge-only routing when the external route disappears

For swap UX, treat Dexie as a route with external constraints, not as a transparent extension of local pool math.

---

## Existing Repo Touchpoints

Relevant local files:

- [projects/chia-cfmm/src/lib/dexieSwap.ts](../../projects/chia-cfmm/src/lib/dexieSwap.ts)
- [projects/chia-cfmm/src/lib/aggregator/dexieAdapter.ts](../../projects/chia-cfmm/src/lib/aggregator/dexieAdapter.ts)
- [projects/chia-cfmm/src/lib/aggregator/execution.ts](../../projects/chia-cfmm/src/lib/aggregator/execution.ts)
- [docs/quests/build-forge-swap-execution-path.md](../quests/build-forge-swap-execution-path.md)

The current repo already has real Dexie execution-path work. This skill is here to preserve the mental model and the helpful upstream links, not to replace those files.

---

## Existing Chia LP Context

As of the latest verified research slice:

- Dexie is a live public source of Chia liquidity discovery
- Dexie references AMM-sourced liquidity as part of its broader venue picture
- Dexie is useful both for quotes and for operator awareness of where external CAT liquidity exists

Treat it as a verified external source for price and route intelligence, but keep pool-state assumptions in Forge and Tibet-specific skills.
