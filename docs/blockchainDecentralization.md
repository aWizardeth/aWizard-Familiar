# Skill: Blockchain & Decentralization Philosophy

> aWizard's understanding of Chia blockchain primitives, the transparency revolution, and why trustless design matters.

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

### BLS Signatures
- Chia uses BLS12-381 aggregate signatures
- Multiple signatures can be aggregated into a single signature
- Used for: NFT minting authority, bond co-signing, battle result attestation

### DID (Decentralized Identity)
- Each wizard has a DID on Chia
- NFTs are bound to DIDs (soulbound) — can't be transferred to another identity
- DIDs can own multiple NFTs representing battle records, achievements, titles

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

### Deterministic Fairness
- AI opponents use **seeded RNG** — given the same seed, the same battle plays out identically
- Seeds are committed before battle starts (commit-reveal scheme)
- Any observer can verify the AI played fairly by replaying the seed

### Bond Escrow
- PvP battles require both players to lock XCH in a bond contract
- The bond is released to the winner (or split on draw) by on-chain settlement
- Neither player can grief — the bond contract enforces the rules

## How aWizard Uses This Knowledge

When a developer asks about:
- **"How does battle verification work?"** → Explain state channels + commit-reveal seeds
- **"Can NFTs be traded?"** → No — soulbound by design, explain why
- **"What happens if someone disconnects mid-battle?"** → State channel force-close mechanism
- **"How are leaderboards trustless?"** → On-chain APS records, anyone can verify

## Source References
- `arcane-battle-protocol/ARCHITECTURE.md` — protocol-level design
- `arcane-battle-protocol/contracts/` — Chialisp smart contracts
- `arcane-battle-protocol/state-channel/` — state channel implementation
- `arcane-battle-protocol/pvp/bond_handler.md` — PvP bond mechanics
- `arcane-battle-protocol/gyms/arcane-bow/ai_seed_model.md` — deterministic AI fairness
