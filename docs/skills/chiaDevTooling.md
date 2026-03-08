# Skill: Chia Dev Tooling

> aWizard's practical reference for the Chia developer docs, debugging tools,
> wallet RPC utilities, packaging infrastructure, and operator workflows that support
> current quests and future backlog work.

---

## Domain

aWizard can guide use of:

- **Developer Guides hub** — where to start and what to read next
- **Tool repos** — what each Chia-maintained utility is good for
- **Wallet and node RPC tooling** — when a synced wallet or full node is required
- **Debugging lineage** — tracing parents, children, and announcements
- **Ops automation** — config-driven database and packaging workflows

## Recommended Reading Order

Use the Chia docs in this order when shaping a new quest or protocol backlog item:

1. `https://docs.chia.net/dev-guides-home/`
   - crash course for state, signatures, inner puzzles, smart coins
2. `https://docs.chia.net/guides/primitives/`
   - NFTs, CATs, offers, DataLayer, clawback, DAOs, VCs, gaming
3. `https://docs.chia.net/guides/tutorials/`
   - simulator, custom puzzles, RPC spend flows, WalletConnect, app structure

This sequence is better than jumping straight into implementation repos with no context.

## Tool Map

### `chia-toolbox`

Status:
- archived, but still useful as a lightweight reference

Most concrete current value:
- offer operations tooling
- cancel offers by CAT asset ID, NFT launcher ID, or all offers
- wallet RPC pagination via `get_all_offers`
- dry-run support before cancellation

Use it when:
- testing marketplace cleanup flows
- operating a wallet with many pending offers
- building small wallet-ops utilities around existing RPC endpoints

### `coin-tracing-scripts`

Purpose:
- trace direct and indirect coin parents and children by inspecting announcement conditions

What it teaches:
- how `CREATE_COIN_ANNOUNCEMENT` pairs with `ASSERT_COIN_ANNOUNCEMENT`
- how `CREATE_PUZZLE_ANNOUNCEMENT` pairs with `ASSERT_PUZZLE_ANNOUNCEMENT`
- how to reconstruct related spends in the same transaction block
- how to read lineage from actual chain data instead of guessing

Operational constraints:
- requires a synced full node on the same machine
- uses Python condition parsing that is less hardened than the Rust path
- malicious spends may cause excessive memory use, so treat it as a developer/debug tool

Use it when:
- a quest spend has confusing inputs or outputs
- a singleton or CAT unwind needs forensic review
- a contract interaction must be explained coin-by-coin

### `database-manager`

Purpose:
- manage users and databases in a cluster from YAML config

Useful patterns:
- `validate` before `apply`
- `ENV:` prefix expansion for deployment-time secrets
- explicit user/database definitions with config validation
- host restriction defaults that fall back to `localhost`
- separate creation of databases, users, read grants, and write grants

Use it when:
- a quest service needs repeatable database setup
- shared staging/prod access must be auditable
- infra should be config-driven instead of manually clicked together

### `build-wheels`

Purpose:
- automate Python wheel building for `pypi.chia.net`

Useful patterns:
- package build automation in CI
- Windows-aware wheel handling
- consistent release artifact generation for Chia-related Python tooling

Use it when:
- internal scripts need to become maintained packages
- quest tooling must be installed reproducibly across machines

## Working Rules

When using Chia tooling, assume:
- wallet RPC tools need a running synced wallet
- tracing tools need a running synced full node
- Python helpers often pin specific `chia-blockchain` versions
- simulator-first validation is the safest path before testnet or mainnet use

## Quick Decision Guide

- Need conceptual grounding: start at the docs hub
- Need asset primitive details: use the primitives index
- Need app flow examples: use the tutorials index
- Need to inspect weird spend relationships: use coin tracing scripts
- Need to cancel or inspect many offers: use `chia-toolbox`
- Need reproducible DB setup: use `database-manager`
- Need repeatable package publishing: use `build-wheels`

## Source References
- `https://github.com/Chia-Network/chia-toolbox`
- `https://github.com/Chia-Network/coin-tracing-scripts`
- `https://github.com/Chia-Network/build-wheels`
- `https://github.com/Chia-Network/database-manager`
- `https://docs.chia.net/dev-guides-home/`
- `https://docs.chia.net/guides/primitives/`
- `https://docs.chia.net/guides/tutorials/`