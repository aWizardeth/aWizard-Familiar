# Skill: Chia Primitives Patterns

> aWizard's working knowledge of singleton state, CAT issuance/admin flows,
> secure distribution patterns, and the canonical Chia primitives that matter for
> quests, campaign systems, and future protocol backlog items.

---

## Domain

aWizard can guide:

- **Singleton state machines** — long-lived on-chain objects with evolving state
- **CAT issuance** — TAIL launch, eve coin handling, issuance flow
- **CR-CAT design** — authorized providers, proofs checking, restricted transfer logic
- **Secure distribution** — secure-the-bag style tree distribution and unwind flows
- **Offer-aware asset design** — how CAT/NFT asset lifecycles interact with offers

## Singleton Pattern

The `workshop-singleton-example` repo is archived, but it still shows the core shape
of a Chia singleton workflow clearly:

1. create a launcher or eve-style root coin
2. curry the current state into the singleton puzzle
3. spend the current singleton to create the next singleton
4. find current state by following the latest child coin
5. recover prior state by reading the previous spend's solution

The example specifically demonstrates:
- creating an initial singleton-backed message object
- synchronizing by walking child coins from a known root coin id
- reading the current message from the previous spend solution
- updating singleton state with a new message and a fee spend

Use this pattern for:
- quest ledgers
- world state checkpoints
- season controllers
- vaults and campaign managers
- long-lived battle or tournament state

## CAT Admin Pattern

The CAT admin tool is the best direct reference here for operational CAT flows.

### Core commands

- `cats`
  - launch a CAT from a TAIL
  - curry TAIL arguments
  - provide solution and destination address
  - optionally aggregate extra signatures and extra spends
- `secure_the_bag`
  - create a bag/tree distribution root from a CSV of targets
  - useful for wide asset distribution planning
- `unwind_the_bag`
  - unwind secured CAT distributions back toward specific targets or leaf spends

### Important CAT concepts

- the CAT asset id is the tree hash of the curried TAIL
- the eve coin is the first CAT coin after issuance and is operationally important
- spending CATs requires correct lineage proof handling
- wallet RPC integration is central to funding and pushing valid transactions

## Credential Restricted CATs

The CAT admin tool also shows the modern CR-CAT pattern:
- `authorized-provider` defines trusted DID providers
- `proofs-checker` or `cr-flag` defines proof validation policy
- transfer rights become policy-aware instead of purely possession-based

This matters for backlog ideas where quest assets, passes, or gated currencies should
only move between approved identities or credential holders.

## Secure The Bag Pattern

`secure_the_bag` is worth keeping in mind for large distribution events:
- targets are read from CSV
- outputs are arranged into a tree with configurable leaf width
- a single root can represent a large preplanned distribution set

Why it matters:
- event airdrops
- reward campaigns
- staged unlock systems
- precomputed distribution trees for community or guild rewards

## Unwind Pattern

`unwind_the_bag` shows the reverse operational reality:
- unwinding spends may need batching to avoid `COST_EXCEEDS_MAX`
- sequential unwinds are required when later spends depend on freshly created children
- fee planning matters because multiple unwind spends may be required
- lineage proof correctness is mandatory during unwind operations

This is useful when thinking through redemption trees, staged claims, or mass
distribution rollback/recovery procedures.

## Design Heuristics

- Use a singleton when state must persist through many spends
- Use a CAT when the asset needs a distinct fungible identity and issuance policy
- Use CR-CAT when ownership alone is not enough and credentials must gate transfer
- Use secure distribution trees when many recipients are known ahead of time
- Use offers when the asset should participate in native atomic trading flows

## Source References
- `https://github.com/Chia-Network/workshop-singleton-example`
- `https://github.com/Chia-Network/CAT-admin-tool`
- `https://docs.chia.net/guides/primitives/`
- `https://docs.chia.net/dev-guides-home/`