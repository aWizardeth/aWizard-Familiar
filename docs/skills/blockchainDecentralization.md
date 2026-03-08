# Skill: Blockchain, Primitives & Decentralization

> aWizard's understanding of Chia blockchain primitives, why trustless design
> matters, and how the core Chia developer guides map onto real quest and game work.

---

## The Transparency Revolution

> Knowledge and power through transparency — security, decentralization, safety, and magic.

aWizard believes:

1. **Transparency breeds trust** — every battle result, every NFT mint, every bond settlement is verifiable on-chain
2. **Decentralization distributes power** — no single server controls outcomes; the blockchain is the referee
3. **Security is non-negotiable** — BLS signatures, cryptographic commitments, and deterministic seeds ensure fairness
4. **Safety through design** — soulbound NFTs can't be stolen; state channels protect mid-battle state; bonds are mutual
5. **Magic is just well-designed systems** — what looks magical to users is rigorous engineering underneath

## Chia Blockchain Primitives

### Coin Model (UTXO)
- Chia uses a **coin set model** (like Bitcoin's UTXO, unlike Ethereum's accounts)
- Every coin is defined by: `(parent_coin_id, puzzle_hash, amount)`
- Coins are spent by providing a **reveal** (the full puzzle) and a **solution**
- Spent coins produce **conditions** that create new coins

### Chialisp
- Chia's on-chain language — a Lisp dialect
- Pure functional, no side effects, no loops (recursion only)
- Compiles to CLVM (Chia Lisp Virtual Machine) bytecode
- Contracts in this project: `bond_contract.clsp`, `gym_contract.clsp`, `nft_contract.clsp`

### Singletons
- Singletons are the canonical Chia primitive for persistent state across spends
- They evolve by creating a single successor coin each time they are updated
- A common pattern is:
	- create an eve coin
	- follow the latest child coin to find current state
	- read prior state from the previous spend's solution
	- spend current singleton to produce the next singleton state
- This is the right primitive for quest ledgers, campaign state, seasonal progression,
	vault-like controllers, and long-lived game objects

### Announcements
- `CREATE_COIN_ANNOUNCEMENT` and `ASSERT_COIN_ANNOUNCEMENT` link multiple spends
- `CREATE_PUZZLE_ANNOUNCEMENT` and `ASSERT_PUZZLE_ANNOUNCEMENT` are the puzzle-hash variant
- Announcement tracing is critical for understanding complex multi-coin transactions,
	wallet behavior, and secure multi-spend coordination
- Parent/child tracing scripts are especially useful when reverse-engineering how a spend assembled its inputs and outputs

### BLS Signatures
- Chia uses BLS12-381 aggregate signatures
- Multiple signatures can be aggregated into a single signature
- Used for: NFT minting authority, bond co-signing, battle result attestation

### DID (Decentralized Identity)
- Each wizard has a DID on Chia
- NFTs are bound to DIDs (soulbound) — can't be transferred to another identity
- DIDs can own multiple NFTs representing battle records, achievements, titles

### CATs, Offers, and NFTs
- CATs are first-class Chia asset tokens governed by a TAIL program
- Offers are Chia's native settlement primitive for atomic trading
- NFTs can be DID-aware, royalty-bearing, airdropped, and batch-minted
- Credential Restricted CATs introduce policy-gated transferability through authorized providers and proofs checking

### Gaming Primitive Caveat
- Chia's gaming documentation is explicitly in alpha and subject to breaking changes
- Treat it as a design reference for development and testing, not as a finished production framework

## Trustless Design Principles

### State Channels (Mini-Eltoo)
- Battles happen **off-chain** in state channels for speed
- Only the final state is committed on-chain
- Either party can force-close by submitting the latest signed state
- Prevents cheating: both players sign every state transition

### Soulbound Identity
- NFTs are **non-transferable** — they represent earned achievements
- Prevents pay-to-win: you can't buy someone else's battle record
- Identity is persistent across all Arcane BOW games

This is a project policy layered on top of Chia primitives, not a network default.

### Deterministic Fairness
- AI opponents use **seeded RNG** — given the same seed, the same battle plays out identically
- Seeds are committed before battle starts (commit-reveal scheme)
- Any observer can verify the AI played fairly by replaying the seed

### Bond Escrow
- PvP battles require both players to lock XCH in a bond contract
- The bond is released to the winner (or split on draw) by on-chain settlement
- Neither player can grief — the bond contract enforces the rules

### Observable State
- Chia's coin model makes state transitions inspectable coin-by-coin
- That supports better audits, easier replay, and more explainable quest progression
- When debugging, follow the coin lineage, announcements, and successor singleton state rather than treating the transaction as opaque

## Developer Guide Priorities

When learning or designing backlog items, work in this order:

1. **Crash course**
	- intro to developing on Chia
	- smart coins
	- signatures
	- state and inner puzzles
2. **Primitives**
	- NFTs
	- CATs
	- offers
	- clawback
	- DAOs
	- verifiable credentials
	- gaming
3. **Tutorials**
	- application structure
	- custom puzzle lock
	- coin spend via RPC
	- simulator usage
	- WalletConnect

That path lines up with the actual needs of this repo: stateful contracts, wallet RPC,
quest assets, and user-facing game flows.

## How aWizard Uses This Knowledge

When a developer asks about:
- **"How does battle verification work?"** → Explain state channels + commit-reveal seeds
- **"Can NFTs be traded?"** → No — soulbound by design, explain why
- **"What happens if someone disconnects mid-battle?"** → State channel force-close mechanism
- **"How are leaderboards trustless?"** → On-chain APS records, anyone can verify
- **"How do we persist quest state?"** → Prefer singleton or lineage-based state machines
- **"How do we debug a weird spend?"** → Trace parents, children, and announcements
- **"Can we gate asset use by identity or credentials?"** → Explain DID and CR-CAT patterns

## Source References
- `arcane-battle-protocol/ARCHITECTURE.md` — protocol-level design
- `arcane-battle-protocol/contracts/` — Chialisp smart contracts
- `arcane-battle-protocol/state-channel/` — state channel implementation
- `arcane-battle-protocol/pvp/bond_handler.md` — PvP bond mechanics
- `arcane-battle-protocol/gyms/arcane-bow/ai_seed_model.md` — deterministic AI fairness
- `https://docs.chia.net/dev-guides-home/` — primary Chia guide hub
- `https://docs.chia.net/guides/primitives/` — primitive reference index
- `https://docs.chia.net/guides/tutorials/` — simulator, RPC, WalletConnect tutorials
- `https://docs.chia.net/guides/gaming/` — Chia gaming references and caveats
