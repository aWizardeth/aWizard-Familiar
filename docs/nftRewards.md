# Skill: NFT & Rewards

> aWizard's knowledge of Magic BOW soulbound NFTs — schema, minting, metadata updates, burn-and-upgrade, and ability tracking.

---

## Domain

aWizard can explain, inspect, and guide development of:

- **NFT Schema** — the on-chain metadata structure
- **Soulbound enforcement** — why transfers are disabled
- **Mint flow** — first-battle NFT creation
- **Update flow** — wins/losses/APS metadata changes
- **Burn & upgrade** — tier evolution (destroy + re-mint)
- **Ability tracking** — teleport, spell_boost unlocks

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
  last_battle_timestamp: string;  // ISO 8601
  soulbound: true;
}
```

## Soulbound Constraint

Enforced at Chia Lisp contract level:
- Standard wallet-to-wallet transfer: **DISABLED**
- Allowed operations:
  - Burn (by owner, for tier evolution only)
  - Contract-authorized metadata update
  - Contract-authorized re-mint (tier upgrade)

## Reward Logic

```
After battle → check if NFT exists:
  No NFT → MINT new one (even on loss — first battle card)
  Has NFT:
    Tier changed? → BURN old + MINT upgraded
    Same tier → UPDATE metadata (wins, losses, APS)
```

## Source References
- `arcane-battle-protocol/gyms/arcane-bow/rewards.md` — reward module spec
- `arcane-battle-protocol/nft/nft_schema.ts` — TypeScript schema
- `arcane-battle-protocol/nft/nft_manager.ts` — mint/update/burn logic
- `arcane-battle-protocol/contracts/nft_contract.clsp` — Chialisp contract
