# Quest: Bootstrap Portal Arbitrage — `portal.awizard.dev`

> **Zero overlap guarantee:**
> - `audit-code-quality.md` → cross-project audit, no new code ✅ separate
> - `bootstrap-aggregator-dex.md` → `projects/chia-aggregator/` ✅ separate
> - `build-chia-multisig-interface.md` → multisig project ✅ separate
> - `scaffold-ecosystem-faucet.md` → `projects/chia-faucet/` ✅ separate
> - **This quest** → `projects/chia-portal/` only

---

## Objective

Build the **Portal Arbitrage Monitor** at `portal.awizard.dev` — a real-time price mismatch
detector and arbitrage UI across all Chia DEX liquidity sources.

**Phase 8 from the DeFi roadmap.**

When the Forge AMM prices diverge from Dexie offer book prices, a profitable atomic swap
opportunity exists. The Portal surfaces these opportunities, shows the expected profit, and
stubs the spend bundle builder that would execute the trade.

This is a **website** — pure frontend, no Discord, no wallet required to view.

---

## Why Now

- ✅ `cfmm.ts` math library reusable for Forge price calculations
- ✅ Aggregator DEX quest establishes the pattern for multi-source price comparison
- Portal operates on `projects/chia-portal/` — fully isolated new codebase
- No contracts needed — price monitoring + UI only this phase, bundle stubs wired later
- Natural complement to the aggregator: aggregator = best swap, portal = best arb

---

## Core Concepts

### Price Mismatch = Arb Opportunity

```
Forge pool:  1 LOVE = 0.010 XCH  (AMM spot price)
Dexie offer: 1 LOVE = 0.012 XCH  (someone selling LOVE cheaper than market)

Arb:  buy LOVE on Dexie for 0.010 XCH → sell on Forge for 0.012 XCH
Net: +0.002 XCH per LOVE (minus fees)
```

### Profitability Filter

Only surface opportunities where `netProfitBps > MIN_PROFIT_BPS` (default 20 bps = 0.2%),
meaning gross spread minus Forge fee (0.3%) minus Dexie spread > threshold.

---

## Steps

### Step 1 — Scaffold `projects/chia-portal/`

Copy from `chia-cfmm` template:

```powershell
robocopy projects/chia-cfmm projects/chia-portal /E `
  /XD node_modules dist .git `
  /XF "*.lock" /NP /NFL /NDL

# package.json: name = "chia-portal"
# vite.config.ts: port = 5177
# Replace App.tsx with portal shell
```

**Port:** 5177 (bank=5175, perps=5174)

---

### Step 2 — Types (`src/lib/types.ts`)

```typescript
export const PRECISION = 1_000_000_000_000n;
export const MOJOS_PER_XCH = 1_000_000_000_000n;

export type PriceSource = 'forge' | 'dexie' | 'tibet' | 'splash';

/** A live price quote from a single source */
export interface PricePoint {
  source: PriceSource;
  pair: string;           // e.g. 'LOVE/XCH'
  bid: bigint;            // best buy price (PRECISION-scaled XCH per token)
  ask: bigint;            // best sell price
  mid: bigint;            // (bid + ask) / 2
  liquidityXch: bigint;   // depth available at this price
  updatedAt: number;      // unix timestamp
}

/** A detected arbitrage route */
export interface ArbOpportunity {
  id: string;
  pair: string;
  buySource: PriceSource;
  sellSource: PriceSource;
  buyPrice: bigint;        // PRECISION-scaled
  sellPrice: bigint;
  spreadBps: number;       // gross spread in basis points
  feesBps: number;         // total fees (buy + sell side)
  netProfitBps: number;    // spreadBps - feesBps
  maxSizeXch: bigint;      // max trade size before slippage exceeds profit
  estimatedProfitXch: bigint; // at maxSizeXch
  detectedAt: number;      // unix timestamp
  status: 'live' | 'stale' | 'captured';
}

/** Historical arb capture record */
export interface ArbCapture {
  opportunityId: string;
  pair: string;
  profitXch: bigint;
  executedAt: number;
  txId?: string;           // spend bundle ID once real execution wired
}
```

---

### Step 3 — `arbMath.ts` (`src/lib/arbMath.ts`)

Pure functions — all bigint, no HTTP, fully typed:

```typescript
// detectOpportunity(prices: PricePoint[], pair: string): ArbOpportunity | null
//   Find the lowest ask across all sources and highest bid → if spread > fees → opportunity
//
// calcNetProfit(spreadBps, forgeFeeBps, dexieSpreadBps): number
//   netProfit = spreadBps - forgeFeeBps - dexieSpreadBps
//
// calcMaxTradeSize(liquidity: bigint, netProfitBps: number): bigint
//   max size limited by available liquidity and slippage
//
// calcEstimatedProfit(size: bigint, netProfitBps: number): bigint
//   size * netProfitBps / 10000
//
// filterProfitable(opps: ArbOpportunity[], minBps: number): ArbOpportunity[]
//   filter to only opportunities above threshold
//
// rankByProfit(opps: ArbOpportunity[]): ArbOpportunity[]
//   sort descending by estimatedProfitXch
```

---

### Step 4 — Frontend Components

#### `OpportunityFeed.tsx`
The main panel — live list of arb opportunities:
- Auto-refreshes mock data every 10 seconds (mock only — real polling Phase 2)
- Each row: Pair | Buy Source | Sell Source | Spread | Net Profit | Max Size | Est. Profit | Action
- Colour coding: net profit bps → 🟢 >50 / 🟡 20–50 / dimmed <20
- "⚡ Execute" button stub → alert with TODO Phase 2 spend bundle
- "Live" badge with pulsing green dot when feed is active
- Empty state: "No profitable opportunities detected — market is efficient 🧹"

#### `PriceCompareTable.tsx`
Side-by-side price comparison across all 4 sources for every tracked pair:
- Rows: LOVE/XCH, SPROUT/XCH, CASTER/XCH, SPELL/XCH, POWER/XCH, HODL/XCH
- Columns: Pair | Forge | Dexie | Tibet | Splash | Best Buy | Best Sell | Spread
- Highlight the best (lowest) ask green, worst red per row
- Price freshness: show age in seconds; grey out if > 60s

#### `CaptureHistory.tsx`
Historical arb captures (mock data):
- Table: Time | Pair | Buy | Sell | Profit | Tx Link
- Running total: "Total arbitrage revenue captured: X XCH"
- "Revenue feeds the insurance fund" footnote
- Mock: 5 historical captures with small realistic profits

#### `PortalStats.tsx`
Header stat bar:
- Opportunities detected (lifetime)
- Opportunities captured (lifetime)
- Total profit XCH
- Avg spread bps across all live pairs
- Insurance fund balance (stub)

---

### Step 5 — App Shell

3 tabs:
- **Opportunities** — `OpportunityFeed` (default)
- **Prices** — `PriceCompareTable`
- **History** — `CaptureHistory`

Persistent `PortalStats` bar below header.

Header: `⚡ Portal — portal.awizard.dev · testnet11`

No wallet connection required to view — connect optional for execute.

---

### Step 6 — TypeScript + Build Check

```powershell
cd projects/chia-portal
npx tsc --noEmit   # target: 0 errors
npm run build      # target: clean dist/
```

---

## Success Criteria

| Criterion | Target |
|-----------|--------|
| `tsc --noEmit` | 0 errors |
| `npm run build` | clean dist |
| `arbMath.ts` | 6 pure functions, no `any`, all bigint |
| OpportunityFeed | Live list, colour-coded, mock refresh |
| PriceCompareTable | 6 pairs × 4 sources grid |
| CaptureHistory | 5 mock rows + running total |
| PortalStats | 5 stat cards in persistent bar |
| Port | 5177 |

---

## Technical Notes

- **PRECISION = 1_000_000_000_000n** — same as all projects
- **Forge fees:** 30 bps (0.3%) — from `cfmm.ts` FEE_BPS constant
- **Dexie spread:** modelled as 5–15 bps (taker spread on offer book)
- **Tibet fees:** 7 bps (their actual testnet constant)
- **Splash fees:** 30 bps assumed
- **Min profit threshold:** 20 bps after all fees — configurable constant
- **No real HTTP calls** this phase — all prices from mock constants with slight jitter
- **Mock jitter:** `price + (Math.random() - 0.5) * price / 100` for realism — wrap in `useMemo` or recalc on interval

---

## Parallel Safety

| Active Quest | Codebase | Overlap |
|---|---|---|
| `audit-code-quality` | cross-project read-only | ❌ none |
| `bootstrap-aggregator-dex` | `projects/chia-aggregator/` | ❌ none |
| `build-chia-multisig-interface` | new multisig project | ❌ none |
| `scaffold-ecosystem-faucet` | `projects/chia-faucet/` | ❌ none |
| **This quest** | `projects/chia-portal/` | ✅ isolated |
