# Skills Index

This file is the authoritative human-readable routing index for `docs/skills/`.

Use it to decide which 1-3 skills to load for a quest instead of duplicating large skill tables in agent docs.
For compact programmatic routing, see `docs/skills/manifest.json`.

This index maps the skill library in `docs/skills/` to the kinds of work the repo is
actually doing: Chia protocol design, DeFi product work, Nightspire frontend flows,
and quest planning.

Use this file when:
- drafting or refining backlog quests
- choosing which domain docs to read before implementation
- deciding whether a problem is protocol, product, UX, or infrastructure

---

## Fast Paths

### Chia Protocol, Contracts, and Assets

| Skill | Use it for |
| --- | --- |
| `blockchainDecentralization.md` | Core Chia primitives, trustless design, singletons, announcements, learning order |
| `chiaPrimitivesPatterns.md` | Singleton patterns, CAT issuance, CR-CATs, secure-the-bag, asset architecture |
| `chiaWalletSdk.md` | Standard lower-level wallet-engine reference for signer behavior, spend construction, and Sage-under-the-hood work |
| `chiaDevTooling.md` | Chia docs hub, tracing tools, RPC tooling, `chia-wallet-sdk`, and package and ops utilities |
| `chiaPerpetuals.md` | Perps market design, margin, funding, liquidation, oracle architecture |
| `nftRewards.md` | Chia NFT minting patterns, DID/royalties, reward collection design |
| `bondPvpEconomy.md` | Escrow, mutual signing, PvP settlement economics |

### Frontend, Wallet, and User Flows

| Skill | Use it for |
| --- | --- |
| `bowAppReference.md` | WalletConnect, CHIP-0002, state channels, tracker patterns, battle state, multi-address scanning |
| `sageRpc.md` | Sage RPC address derivations, `increase_derivation_index`, gap-limit scan, operator patterns |
| `discordActivityAuth.md` | Discord Activity auth, OAuth2, embedded app flow |
| `networkGameplayUX.md` | Loading states, spell-cast UX, network feedback, multiplayer responsiveness |
| `nightspireTheme.md` | Canonical Nightspire design language, tokens, component styling |
| `projectArchitecture.md` | File placement, naming, boundaries, module organization |
| `deploymentInfra.md` | Vercel/VPS deployment, env boundaries, DB and release tooling |

### Game Systems and World Design

| Skill | Use it for |
| --- | --- |
| `battleKnowledge.md` | Battle flow, turn systems, engine logic |
| `aiSeedModel.md` | Deterministic AI, seeded RNG, fairness model |
| `apsTierSystem.md` | Progression, ranks, unlock thresholds |
| `leaderboardRankings.md` | Ranking logic, score surfaces, trustless record planning |
| `tournamentSystem.md` | Brackets, seeding, prize logic |
| `snesWorldEngine.md` | World navigation, map systems, encounter integration |

### Quest and Planning Workflow

| Skill | Use it for |
| --- | --- |
| `questManagement.md` | Backlog workflow, foundation-first delivery, enhancement quest pattern |

---

## Suggested Reading by Quest Type

### DeFi / Protocol Quest
- `blockchainDecentralization.md`
- `chiaPrimitivesPatterns.md`
- `chiaDevTooling.md` — only if tooling/Sage internals are in scope
- `deploymentInfra.md` — only if deployment is in scope

### Wallet / Signing / Multisig Quest
- `bowAppReference.md` — WalletConnect, CHIP-0002, multi-address scanning
- `sageRpc.md` — operator/backend address derivation, gap-limit expansion
- `chiaWalletSdk.md` — only when dropping below the Sage/WC layer into signer behavior

Use `bowAppReference.md` first for all WalletConnect and CHIP-0002 UI flows, including `chip0002_getPublicKeys` multi-address patterns.
Use `sageRpc.md` for backend/operator flows that call Sage RPC directly (`/get_derivations`, `/increase_derivation_index`).
Use `chiaWalletSdk.md` only when the quest requires raw spend construction or wallet engine internals below Sage.
Only add `blockchainDecentralization.md` + `chiaPrimitivesPatterns.md` if the quest also touches protocol design or asset architecture.

### NFT / Rewards / Collections Quest
- `nftRewards.md`
- `blockchainDecentralization.md`
- `chiaPrimitivesPatterns.md`
- `deploymentInfra.md`

### Game / Nightspire Quest
- `battleKnowledge.md`
- `networkGameplayUX.md`
- `nightspireTheme.md`
- `discordActivityAuth.md`

### Architecture / Refactor Quest
- `projectArchitecture.md`
- `deploymentInfra.md`
- `bowAppReference.md`

---

## Indexes and Companion Docs

- `docs/ARCHITECTURE_INDEX.md` — where to find the right architecture document
- `docs/ARCHITECTURE.md` — ecosystem-level product and subdomain map
- `docs/QUEST_WORKFLOW.md` — quest lifecycle and backlog movement rules

## Backlog Wiring Rule

New backlog quests should include:
- a short `Reference Indexes` section linking this file and `docs/ARCHITECTURE_INDEX.md`
- a short `Relevant Skills` section listing the few skills that matter most for the quest

This keeps planning docs actionable instead of becoming disconnected idea dumps.