# Skill: Tibet UI Frontend and Wallet Flow Reference

> Frontend-facing knowledge preserved from the Tibet UI repository.
> Use this when designing external swap UX, pair discovery, or wallet flows for Chia DEX integrations.

---

## Domain

This skill is for UI and product patterns learned from Tibet UI:

- pair discovery and route entry points
- quote display behavior
- request caching expectations
- frontend wallet flow ideas
- what a public Chia DEX frontend needs to communicate clearly

Use it for:

- Forge swap or aggregator UI work
- external venue route presentation
- Chia DEX wallet and quote UX
- preserving Tibet UI repo references in the skill library

---

## Upstream Reference

- https://github.com/yakuhito/tibet-ui

The repository itself is the useful artifact here. Its README is minimal, but the repo structure and recent commit history still make it worth indexing.

---

## Verified Repo Signals

Useful signals visible from the upstream repo:

- Next.js application
- TypeScript-heavy frontend
- SWR was intentionally added for request handling and caching
- active recent maintenance around pair fetching behavior
- `src/` is the main reference area for UI patterns

This is enough to keep Tibet UI in the skill library as a frontend reference, even though the README itself is sparse.

---

## Practical UX Lessons

### Pair discovery matters

External DEX UI cannot assume the user already knows the exact pair id. A good Chia DEX frontend needs:

- token-aware route entry points
- resilient pair fetching
- sensible fallback behavior when pair data is missing or late

### Request caching matters

The explicit addition of SWR upstream is a useful signal:

- quotes and pair state should be cached briefly
- refetches should be deliberate, not noisy
- pair metadata and reserve views should avoid unnecessary churn

### External routes need strong labeling

For Forge UI, Tibet-style or Dexie-style routes should always say what venue the user is about to use. Do not present an external quote as if it were a native Forge pool operation.

### Wallet feedback matters

External venue UX should clearly separate:

- quote ready
- waiting for wallet
- signed locally
- submitted externally
- route expired or changed

---

## How to Reuse This in Forge

Use Tibet UI as a reference for:

- route cards for external venues
- pair-first navigation patterns
- cache strategy for external liquidity metadata
- separating quote state from execution state

Do not copy its exact stack decisions blindly. Use the repo for behavior and flow cues, then adapt to the existing Forge architecture and Nightspire language.
