# Skill: Tibet AMM and Offer-Driven LP Flows

> Repo knowledge for TibetSwap V2 style XCH/CAT AMM mechanics on Chia.
> Use this when reasoning about external AMM pools, LP flows, or offer-settled swap mechanics.

---

## Domain

This skill covers the protocol-side Tibet knowledge most useful to Forge:

- XCH/CAT pair architecture
- singleton and router pattern
- swap, add-liquidity, remove-liquidity, and pair-creation flows
- LP token mint and burn mechanics
- reserve coin handling and flashloan-style reserve control
- how Tibet differs from Forge's pool-controlled LP CAT model

Use it for:

- `tibetAdapter.ts` design
- comparing Forge AMM behavior to existing Chia LP systems
- reasoning about external pool discovery and quote sourcing
- understanding how Tibet turns offers into atomic pool state transitions

---

## Upstream References

- https://github.com/yakuhito/tibet
- https://github.com/yakuhito/tibet/blob/master/testnet.md
- https://github.com/yakuhito/tibet/blob/master/mainnet.md
- https://github.com/yakuhito/tibet/blob/master/SECURITY.md
- https://github.com/yakuhito/tibet/blob/master/TESTING.md
- https://v2.tibetswap.io/

These links should stay in the skill. They are the fastest way to re-open the source of truth.

---

## Core Model

Tibet is an offer-aware XCH/CAT AMM inspired by Uniswap V1, adapted to Chia's coin-set model.

Important characteristics:

- pair state is maintained through a singleton
- pairs are XCH/token, not token/token as the default primitive
- token/token swaps are expected to route through XCH
- offers are used as the settlement guardrail
- only the pair singleton changes during a trade for that pair

This matters because it means different pairs can trade independently in parallel, without a shared router spend on every trade.

---

## Router and Pair Pattern

Tibet uses a router to track deployed pair singletons and keep pair deployment valid.

Key ideas from the upstream README and code:

- the router tracks all pair singletons
- pair launch validity is constrained by the deploy flow
- the initial pair state is expected to start at zero liquidity and zero reserves
- the pair singleton is the stateful center of the pool

Useful upstream entry points:

- `create_pair_with_liquidity`
- `respond_to_swap_offer`
- `respond_to_deposit_liquidity_offer`
- `respond_to_remove_liquidity_offer`
- `/new-pair/{asset_id}` in `api.py`

---

## Pair State and Reserve Handling

The practical Tibet pair state is centered on:

- liquidity supply
- XCH reserve
- token reserve

Reserve handling is notable because Tibet deliberately allows strong control over reserve spends within a single atomic transaction, using a singleton-aware flashloan-style reserve wrapper.

Important source concepts:

- `p2_singleton_flashloan`
- reserve coins are spent and re-created atomically
- the transaction can use reserves freely as long as the final post-transaction reserve amounts are correct

This is one of the most helpful mental bridges between Tibet and Forge: both rely on atomic control of stateful reserve transitions, but they package the authority differently.

---

## Offer-Driven Operations

### Swap

Tibet's swap path responds to a user-created offer rather than asking users to handcraft pool spends directly.

What the driver does:

- parses the incoming Chia offer
- detects the ephemeral offered coin created by that offer
- computes the new reserve amounts
- spends the pair singleton to move state forward
- spends and recreates reserve coins atomically
- returns a spend bundle that satisfies the offer and the AMM transition together

Key upstream function:

- `respond_to_swap_offer`

### Add liquidity

The deposit path similarly responds to an offer and mints liquidity tokens.

Relevant upstream facts:

- deposit flow detects ephemeral XCH and token coins from the offer
- target liquidity scales from reserves when the pool already exists
- liquidity CAT minting is part of the atomic response bundle
- reserve recreation is guarded through announcements and offer-aware settlement conditions

Key upstream function:

- `respond_to_deposit_liquidity_offer`

### Remove liquidity

The remove path burns liquidity CAT and redeems reserves.

Relevant upstream facts:

- the offer provides the ephemeral liquidity CAT input
- burn uses CAT `extra_delta` semantics with a pair-specific liquidity tail
- the pair singleton is respent with updated liquidity and reserve values
- reserve coins are recreated to the new post-withdraw amounts

Key upstream function:

- `respond_to_remove_liquidity_offer`

### Pair creation

Pair creation is not just singleton launch. Tibet can create the pair and seed initial liquidity together.

Key upstream function:

- `create_pair_with_liquidity`

That pattern is relevant to Forge because it shows a truthful upstream example where pair deployment and first-liquidity seeding are intentionally linked.

---

## LP Token Pattern

Tibet uses a per-pair liquidity CAT tail.

Important traits from the source:

- liquidity token identity is derived from pair context
- deposit mints LP via CAT spend construction
- remove burns LP via negative `extra_delta`
- lineage proof correctness is essential for reserve CAT and LP CAT spends

This is useful comparison material for Forge, even though Forge now uses a different pool-controlled LP CAT model and V2 announcement semantics.

Do not assume Tibet's LP CAT authority pattern is identical to Forge's current pool-controlled LP CAT flow.

---

## Economic Notes

Upstream Tibet documentation explicitly frames:

- a 0.7% fee candidate through `inverse_fee=993`
- XCH/token as the default pair model
- token/token swaps as chained XCH hops
- "zap" and chained action ideas as legitimate future extensions

These are useful product comparisons when evaluating aggregator UX or multi-hop routing strategy.

---

## Frontend and Wallet Notes from tibet-ui

The Tibet UI repository is worth keeping alongside the protocol repo:

- https://github.com/yakuhito/tibet-ui

Useful frontend-level reminders from that codebase and repo structure:

- Next.js plus TypeScript frontend
- SWR-based request caching was added for request handling
- pair discovery and token fetch behavior are explicit UI concerns
- recent commit history shows active work around pair fetching by token route

Use tibet-ui as a UI reference for:

- external AMM quote UX
- pair discovery behavior
- wallet-aware swap and LP screens
- how a public Chia DEX presents pool interactions to users

---

## How Tibet Differs from Forge

Treat these as complementary, not interchangeable:

- Tibet is offer-settled first
- Forge's current pool lifecycle work is spend-bundle-first for pool operations
- Tibet centers on XCH/token pairs
- Forge is targeting broader weighted multi-asset pool behavior
- Tibet LP mint and burn logic lives in pair-driven AMM flows
- Forge LP issuance is now tied to the pool-controlled TAIL and current V2 deployment/test path

When building adapters, keep the execution boundary explicit instead of flattening both systems into a fake common model.
