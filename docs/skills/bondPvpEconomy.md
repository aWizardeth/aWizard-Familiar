# Skill: Bond & PvP Economy

> aWizard's knowledge of the bond staking system — proposals, mutual signing, on-chain locking, and winner settlement.

---

## Domain

aWizard can explain and guide:

- **Bond proposals** — creating and signing stake terms
- **Mutual acceptance** — both players must sign before lock
- **On-chain locking** — bond_contract.clsp holds funds
- **Settlement** — winner receives stake after battle
- **Expiry** — auto-forfeit if battle not started by deadline

## Bond Terms Schema

```typescript
interface BondTerms {
  match_id: string;
  player_a: string;          // wallet address
  player_b: string;          // wallet address
  bond_amount_mojos: number; // in XCH mojos
  currency: "XCH" | "CAT";
  expiry_timestamp: number;  // unix — auto-forfeit
  signed_by_a: string;       // signature
  signed_by_b: string;       // signature (null until accepted)
}
```

## Bond Flow

```
Player A creates proposal + signs -> sends to Player B
-> Player B reviews + counter-signs -> mutual acceptance
-> Both signatures present -> lock_bond() submits to chain
-> Battle plays out via state channel
-> Winner determined -> settle_bond() releases funds to winner
```

## Security Properties

- **Mutual consent** — neither player can be locked into a bond without signing
- **Cryptographic verification** — signatures prove agreement
- **On-chain escrow** — funds held by contract, not by either player
- **Expiry protection** — auto-refund if battle never starts

## Source References
- `arcane-battle-protocol/pvp/bond_handler.md` — full implementation
- `arcane-battle-protocol/contracts/bond_contract.clsp` — Chialisp contract
