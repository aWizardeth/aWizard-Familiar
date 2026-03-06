# Quest: Bootstrap Bank of Wizards — `bank.awizard.dev`

**Status:** ACTIVE  
**Started:** 2025-01-XX  
**Champion:** aWizard 🧙  

---

## Objective

Build the **Bank of Wizards** portfolio hub at `bank.awizard.dev` — a unified dashboard that aggregates all assets, positions, and earnings across the aWizard DeFi ecosystem.

**Show the wizard everything they own:**
- XCH / CAT balances
- CFMM LP positions (forge.awizard.dev)
- Treasure Chest NFTs (storefront)
- Perpetuals positions (planned)
- Emoji token holdings (craft.awizard.dev)

---

## Why Now

All core DeFi primitives are complete:
- ✅ `chia-cfmm` — weighted AMM with LP NFTs
- ✅ `chia-treasure-chest` — programmable vaults
- ✅ `chia-craft` — emoji token market

The Bank is the capstone: read-only aggregation with zero new contracts.

---

## Steps Completed

### ✅ Step 1 — Project Scaffold

Created `projects/chia-bank/` from `chia-cfmm` template:
- Port: **5175**
- Vite + React 19 + TypeScript strict
- Nightspire CSS theme preloaded
- WalletConnect CHIP-0002 integration ready

---

## Next Steps

### Step 2 — Core Types & Aggregator

Create `src/lib/types.ts`:
```typescript
export interface AssetBalance {
  assetId: string;     // '' for XCH
  symbol: string;
  emoji?: string;
  balanceMojos: bigint;
  priceXch?: bigint;   // PRECISION-scaled
}

export interface LpPosition {
  nftId: string;
  poolId: string;
  symbol: string;      // 'XCH/LOVE 80/20'
  sharesHeld: bigint;
  valueXch: bigint;
  feesEarnedXch: bigint;
}

export interface TreasureChest {
  nftId: string;
  name: string;
  contentsXch: bigint;
  locked: boolean;
}

export interface PortfolioSummary {
  totalNetWorthXch: bigint;
  assets: AssetBalance[];
  lpPositions: LpPosition[];
  treasureChests: TreasureChest[];
}
```

Create `src/lib/bankAggregator.ts`:
```typescript
import { WalletConnectProvider } from './walletConnect';

export async function fetchPortfolio(
  provider: WalletConnectProvider
): Promise<PortfolioSummary> {
  // Query wallet for all coins
  const address = provider.getAddress();
  
  // Fetch XCH balance
  const xchCoins = await provider.getAssetCoins('');
  
  // Fetch CAT balances (LOVE, SPROUT, etc.)
  const catBalances = await fetchCatBalances(provider);
  
  // Fetch LP NFT positions
  const lpPositions = await fetchLpPositions(provider);
  
  // Fetch treasure chest NFTs
  const treasureChests = await fetchTreasureChests(provider);
  
  // Calculate total net worth
  const totalNetWorthXch = calculateNetWorth({
    xchCoins,
    catBalances,
    lpPositions,
    treasureChests,
  });
  
  return { totalNetWorthXch, assets: [...], lpPositions, treasureChests };
}
```

### Step 3 — Dashboard Components

**`src/components/PortfolioOverview.tsx`:**
- Total net worth in XCH (big number at top)
- Breakdown: XCH | CATs | LP | NFTs
- 7-day change graph (optional)

**`src/components/AssetList.tsx`:**
- Table: Asset | Balance | Price | Value
- Sort by value DESC
- Click → asset detail modal

**`src/components/LpPositionCard.tsx`:**
- Pool name + emoji icons
- Share % of pool
- Current value + fees earned
- "Manage" → opens forge.awizard.dev

**`src/components/TreasureChestCard.tsx`:**
- Chest name + lock status
- Contents value
- Unlock timer if locked
- "View" → opens storefront

### Step 4 — Navigation & Layout

**`src/App.tsx`:**
```tsx
<div className="bank-layout">
  <Header />
  <Sidebar>
    <NavLink to="/overview">Overview</NavLink>
    <NavLink to="/assets">Assets</NavLink>
    <NavLink to="/positions">LP Positions</NavLink>
    <NavLink to="/nfts">Treasure Chests</NavLink>
  </Sidebar>
  <main>
    <Routes>
      <Route path="/" element={<PortfolioOverview />} />
      <Route path="/assets" element={<AssetList />} />
      <Route path="/positions" element={<LpPositionList />} />
      <Route path="/nfts" element={<TreasureChestList />} />
    </Routes>
  </main>
</div>
```

### Step 5 — Deploy & Test

- [ ] Run dev server on `:5175`
- [ ] Connect Sage wallet
- [ ] Verify asset balances display correctly
- [ ] Mock LP positions if no testnet11 pools exist yet
- [ ] Verify UI matches Nightspire theme

---

## Technical Notes

**Read-only aggregation:**
- No spend bundles needed
- All WalletConnect calls are `getAssetCoins` / `getAssetBalance`
- Safe to run before contracts are fully deployed

**Price oracles:**
- Use CFMM spot price for CAT/XCH pairs
- Fallback to manual USD/XCH if needed
- Display both XCH and USD equivalents

**LP position valuation:**
- Query pool reserves from CFMM indexer
- Calculate pro-rata share based on NFT supply
- Include unclaimed fees in total value

---

## Success Criteria

✅ Wallet connects and displays address  
✅ XCH balance shows correctly  
✅ CAT tokens list with emoji icons  
✅ LP positions display with current value  
✅ Treasure Chest NFTs render as cards  
✅ Total net worth updates on refresh  
✅ UI matches Nightspire CSS theme  

---

## Time Estimate

**2-3 hours** — straightforward aggregation and UI composition.

---

*Quest active — let's build the Bank! 🏦✨*
