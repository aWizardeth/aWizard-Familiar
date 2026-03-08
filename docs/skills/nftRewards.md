# Skill: NFT & Rewards

> aWizard's knowledge of Magic BOW NFTs and the practical Chia NFT tooling needed
> to mint, update, bulk-create, and operationalize reward collections.

---

## Domain

aWizard can explain, inspect, and guide development of:

- **Project NFT schema** — the Battle of Wizards reward metadata structure
- **Soulbound enforcement** — why Magic BOW NFTs are intentionally non-transferable
- **Bulk mint workflows** — metadata CSV, spend-bundle generation, submission
- **DID and royalty flows** — when DID-backed NFTs are required or useful
- **Reward lifecycle** — first mint, metadata update, burn-and-upgrade, re-mint
- **Offer creation** — generating sell offers for standard transferable NFTs
- **Simulator testing** — validating mint flows before mainnet deployment

## Important Distinction

Magic BOW reward NFTs are a **project-specific design** and can be made soulbound by
contract choice.

Standard Chia NFTs are not inherently soulbound:
- They can be transferable
- They can be DID-associated
- They can include royalties
- They can be bulk minted and later listed through offers

Use standard Chia NFT behavior as the base primitive, then add soulbound rules only
where the game economy truly requires them.

## NFT Schema

```typescript
interface MagicBOWNFT {
  owner_wallet: string;
  wins: number;
  losses: number;
  arcane_power_score: number;
  tier: "Initiate" | "Adept" | "Archmage" | "Grand Wizard";
  unlocked_abilities: {
    teleport: boolean;
    spell_boost: number;
  };
  last_battle_timestamp: string;
  soulbound: true;
}
```

Recommended mapping for Chia-native metadata planning:
- Stable on-chain fields for edition, tier, lineage, and quest provenance
- Mutable game-state fields tracked off-chain or through controlled re-mint/update flows
- Separate player progression state from static media URIs when possible

## Soulbound Constraint

If Magic BOW keeps soulbound rewards, enforce it deliberately at the contract level:
- Standard wallet-to-wallet transfer: **disabled**
- Allowed operations:
  - Burn by owner when a tier evolution consumes the old NFT
  - Contract-authorized metadata update or replacement
  - Contract-authorized re-mint for upgraded form factors

This should be treated as a game rule, not a default Chia NFT assumption.

## Reward Logic

```
After battle -> check if NFT exists:
  No NFT -> mint new one
  Has NFT:
    Tier changed? -> burn old + mint upgraded version
    Same tier -> update tracked progression fields
```

For backlog design, separate three reward modes:
- **Achievement NFTs**: soulbound, non-tradable proof of progress
- **Drop NFTs**: transferable cosmetics or loot
- **Quest batch NFTs**: bulk-minted campaign or event rewards

## Bulk Minting Pattern

The Chia NFT minting tool establishes a practical two-phase workflow:

1. **Offline creation**
   - Read `metadata.csv`
   - Build mint spend bundles in chunks, commonly 25 NFTs per bundle
   - Support optional target addresses for airdrops
2. **Online submission**
   - Push spend bundles to the mempool
   - Monitor confirmation state
   - Optionally create offer files per minted NFT

Key operational lessons:
- The minting wallet must be fully synced
- DID-backed minting is required for DID-associated flows and royalties
- Bulk minting expects a single funding coin large enough for the batch
- Submission logic may need retries when spends fall out of the mempool
- The process should support resuming from the first unspent bundle

## Metadata CSV Planning

Useful Chia-native metadata fields for quest and reward pipelines:
- `hash`
- `uris`
- `meta_hash`
- `meta_uris`
- `license_hash`
- `license_uris`
- `edition_number`
- `edition_total`
- `target` for airdrops when applicable

Use this structure when planning future event drops, founder rewards, seasonal quest
collections, or achievement campaigns.

## DID, Royalties, and Offers

Standard Chia NFT operations support:
- DID-attached minting
- Royalty address and basis-point royalty percentage
- Offer generation after minting

That means future backlog items can cleanly split into:
- **identity-bound achievements**
- **tradable collectibles**
- **campaign airdrops**
- **sale-ready collections with royalties**

## Testing Pattern

The recommended local validation loop is:
- initialize Chia simulator
- start wallet against simulator root
- create DID if DID-backed minting is used
- run mint tests before pointing at testnet or mainnet

This is the correct path for quest reward systems before real deployment.

## Source References
- `arcane-battle-protocol/gyms/arcane-bow/rewards.md` — reward module spec
- `arcane-battle-protocol/nft/nft_schema.ts` — TypeScript schema
- `arcane-battle-protocol/nft/nft_manager.ts` — mint/update/burn logic
- `arcane-battle-protocol/contracts/nft_contract.clsp` — Chialisp contract
- `https://github.com/Chia-Network/chia-nft-minting-tool` — bulk mint CLI, DID, royalties, offer generation
- `https://docs.chia.net/guides/primitives/` — NFT primitive guidance
- `https://docs.chia.net/guides/tutorials/` — simulator and wallet workflow references
