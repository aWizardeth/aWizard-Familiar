# Skill: Forge LP CAT — Pool-Controlled TAIL System

> Architecture and implementation patterns for Forge's pool-controlled LP CAT TAIL.
> Learned from: `build-forge-pool-controlled-tail` quest (completed 2026-03-30).

---

## Domain

This skill covers the Forge-specific LP CAT TAIL system — where LP token issuance
is governed on-chain by the pool singleton, not a centralized key or one-time genesis spend.

Use it for:
- Deriving LP CAT asset IDs from a pool launcher ID
- Building deposit/withdrawal authority coin spend descriptors
- Computing pool announcement IDs for TAIL authorization
- Understanding the two-contract system (TAIL + authority coin)

---

## Core Architecture

### Two-Contract System

| Contract | File | Role |
|----------|------|------|
| LP CAT TAIL | `forge_lp_cat_tail.rue` | Validates mint/burn, asserts pool announcement |
| Authority Coin | `lp_cat_authority_coin.rue` | 1-mojo persistent coin, recreates on each LP change |

### Security Chain

```
Pool singleton
  └─ CREATE_PUZZLE_ANNOUNCEMENT(tree_hash([recipient_ph, lp_amount, pool_id]))
       │
       ├── ASSERT_PUZZLE_ANNOUNCEMENT ← Authority coin (validates + recreates)
       │     └─ CREATE_PUZZLE_ANNOUNCEMENT(tree_hash([launcher_id, asset_id, new_total_lp]))
       │           │
       │           └── ASSERT_PUZZLE_ANNOUNCEMENT ← LP CAT TAIL (authorizes mint/burn)
       │
       └── ASSERT_PUZZLE_ANNOUNCEMENT ← Reserve coins (lock deposit amounts)
```

No mint or burn of LP CATs is valid without a matching pool singleton announcement.

---

## Announcement Format

### Deposit (pool singleton emits)
```
tree_hash([recipient_puzzle_hash, lp_out, pool_launcher_id])
```

### Withdrawal (pool singleton emits)
```
tree_hash([redeemer_puzzle_hash, lp_burned, pool_launcher_id])
```

### Authority coin self-announcement (emitted when it recreates itself)
```
tree_hash([launcher_id, lp_cat_asset_id, next_total_lp_supply])
```

The TAIL asserts the authority coin's announcement to confirm authority was spent.

---

## Puzzle Hash Derivation

LP CAT `asset_id` = TAIL puzzle hash = deterministic from `launcher_id` alone.

```python
# Python — pool_tail_authority_spend.py
from pool_tail_authority_spend import (
    compute_lp_cat_asset_id,
    compute_authority_puzzle_hash,
)
contracts_dir = Path("contracts/")
asset_id  = compute_lp_cat_asset_id(contracts_dir, launcher_id)
auth_ph   = compute_authority_puzzle_hash(contracts_dir, launcher_id, asset_id)
```

```typescript
// TypeScript — src/lib/lpCatAuthority.ts
const assetId  = await computeLpCatAssetId(launcherId);
const tailPh   = await computeTailPuzzleHash(launcherId);
const authPh   = await computeAuthorityPuzzleHash(launcherId, assetId);
const annId    = await computeDepositAnnouncementId(poolPh, recipientPh, lpOut, poolId);
```

---

## CLI

```bash
# From contracts/ directory
python pool_tail_authority_spend.py --launcher-id <hex>
# Returns: { tail_puzzle_hash, lp_cat_asset_id, authority_puzzle_hash }
```

---

## Authority Coin Spend Pattern

```python
# Deposit descriptor
auth = build_deposit_authority_spend(
    contracts_dir, launcher_id, lp_cat_asset_id,
    authority_coin_parent,   # parent_coin_info of current authority coin
    pool_puzzle_hash,
    recipient_puzzle_hash,
    lp_out,
    current_total_lp,
)
# auth.coin_id                      → spend this coin in the bundle
# auth.authority_announcement_id()  → ID the TAIL must ASSERT_PUZZLE_ANNOUNCEMENT
# auth.to_dict()                    → JSON-serializable descriptor

# Withdrawal descriptor
auth = build_withdraw_authority_spend(
    contracts_dir, launcher_id, lp_cat_asset_id,
    authority_coin_parent,
    pool_puzzle_hash,
    redeemer_puzzle_hash,
    lp_burned,
    current_total_lp,
)
```

The descriptors are now emitted in the `authority_spend` field of `forge_add_liquidity.py`
and `forge_remove_liquidity.py` `--dry-run` output. When the pool singleton puzzle reveal
becomes available, these descriptors wire directly into the full spend bundle.

---

## Key Files

| File | Purpose |
|------|---------|
| `contracts/forge_lp_cat_tail.rue` | TAIL Rue source |
| `contracts/lp_cat_authority_coin.rue` | Authority coin Rue source |
| `contracts/compiled/forge_lp_cat_tail.clvm.hex` | Compiled TAIL CLVM |
| `contracts/compiled/lp_cat_authority_coin.clvm.hex` | Compiled authority CLVM |
| `contracts/pool_tail_authority_spend.py` | Python descriptor builders + CLI |
| `contracts/forge_add_liquidity.py` | Deposit script (authority_spend in result) |
| `contracts/forge_remove_liquidity.py` | Withdraw script (authority_spend in result) |
| `src/lib/lpCatAuthority.ts` | TypeScript puzzle hash derivation |
| `src/lib/spendBundles.ts` | TypeScript `LpAuthorityDescriptor` types + builders |
| `api/pool-tail-authority.js` | GET endpoint: puzzle hashes from launcher ID |

---

## Known Values (TM10 — testnet11)

| Property | Value |
|----------|-------|
| Launcher ID | `6fb445ab73bb90002f294c72dafd4f9f3ae2c2fb5671d0b4ab4b88a290ed3cc0` |
| New LP CAT asset ID (pool TAIL) | `efce9954287878a3f645ddbf76cb75666828b6c7293296dceab69ae66189c62a` |
| Authority puzzle hash | `726fcf284bf6bb1b65959d35a44c561c65490643c6de5fe5a3b63b1b38eef948` |
| Old FLPTM10 asset ID (fixed TAIL) | `418c18a5222ad34812633877747ed253394344d65077bf13e0b84b85494256fb` |

---

## Known Blockers

- **Pool singleton puzzle reveal**: The pool's compiled CLVM is in `dist/` but the Rue source
  is not on disk. Full spend bundle construction (pool singleton spend + reserve spends + authority
  coin + TAIL) cannot proceed without it.
- **Authority coin parent**: Must be set in `authorityCoinParent` in the deployment index batch
  for the authority coin ID to be computed. Without it, puzzle hash derivation still works but
  the full coin ID is unknown.
- **Simulator test**: No full lifecycle test (deposit → verify LP CAT minted) has been run yet.

---

## Related Quests

- `done/build-forge-pool-controlled-tail.md` — foundation quest (completed 2026-03-30)
- `backlog/enhance-forge-full-liquidity-bundle.md` — full bundle construction (next)
