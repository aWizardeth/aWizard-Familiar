# Quest: Bootstrap Chia Aggregator DEX — `swap.awizard.dev`

> **Zero overlap guarantee:**
> - `bootstrap-bank-of-wizards-quest.md` → `projects/chia-bank/` ✅ separate
> - `build-chia-multisig-interface.md` → new multisig project ✅ separate
> - **This quest** → `projects/chia-aggregator/` only

---

## Objective

Build the **smart order router** and unified swap interface for `swap.awizard.dev` — aggregates liquidity across every major Chia DEX to always surface the best rate for the user.

**Phase 7 from the DeFi roadmap.** This is Chia's equivalent of 1inch / ParaSwap.

**Exchanges integrated:**
| Source | Type | Integration |
|--------|------|-------------|
| **Forge (our CFMM)** | Weighted AMM | `cfmm.ts` math, live pool state |
| **Tibetswap** | Constant-product AMM | Public API + pool configs |
| **Dexie** | Offer book | Offer book REST API |
| **Splash** | AMM | Their price API |

---

## Why Now

- ✅ `chia-cfmm` — our own CFMM math library is fully implemented and ready to reuse
- ✅ `chia-craft` — emoji tokens now exist that will need best-price swapping
- The Bank of Wizards (`chia-bank`) will link to this aggregator for inline swaps
- The aggregator earns a 0.05% protocol fee → feeds the insurance fund

---

## Steps

### Step 1 — Scaffold `projects/chia-aggregator/`

Copy from `chia-cfmm` template (proven pattern):

```powershell
robocopy projects/chia-cfmm projects/chia-aggregator /E `
  /XD node_modules dist .git `
  /XF "*.lock" /NP /NFL /NDL

# package.json: name = "chia-aggregator"
# vite.config.ts: port = 5176
# Clear src/lib/cfmm.ts → replaced with router.ts
# Replace App.tsx with aggregator shell
```

**Port:** 5176 (perps=5174, bank=5175)

---

### Step 2 — Router Types

Create `src/lib/types.ts` with aggregator-specific interfaces:

```typescript
export const PRECISION = 1_000_000_000_000n;
export const MOJOS_PER_XCH = 1_000_000_000_000n;

export type DexSource = 'forge' | 'tibetswap' | 'dexie' | 'splash';

/** A price quote from a single DEX source */
export interface DexQuote {
  source: DexSource;
  tokenIn: string;         // asset_id ('' = XCH)
  tokenOut: string;
  amountIn: bigint;        // mojos
  amountOut: bigint;       // mojos
  priceImpactBps: number;  // basis points, e.g. 50 = 0.5%
  feeBps: number;          // protocol fee in bps
  routePath: string[];     // asset_ids of hops if multi-hop
  expiresAt?: number;      // unix ts (offer book quotes expire)
}

/** The winning route recommended to the user */
export interface BestRoute {
  quote: DexQuote;
  savingsVsWorstBps: number;  // how much better than the worst quote
  allQuotes: DexQuote[];
}

/** An aggregated token pair the UI can display */
export interface TokenPair {
  baseAssetId: string;    // '' = XCH
  quoteAssetId: string;
  baseSymbol: string;
  quoteSymbol: string;
  baseEmoji?: string;
  quoteEmoji?: string;
}
```

---

### Step 3 — `router.ts`

Create `src/lib/router.ts` — the core logic. All pure functions, no HTTP calls (mocked in this phase):

```typescript
// findBestRoute(
//   quotes: DexQuote[],
//   amountIn: bigint,
//   tokenIn: string,
//   tokenOut: string
// ): BestRoute
//   → sort by amountOut descending, return winner + savings vs last

// calcForgeQuote(pool, tokenIn, amountIn) → DexQuote
//   → delegate to cfmm.ts calcSwapOut, calcPriceImpactBps

// calcTibetQuote(reserve0, reserve1, amountIn) → DexQuote
//   → standard x*y=k constant product formula

// calcDexieQuote(offerPrice, offerAmount, amountIn) → DexQuote
//   → fill against best offer; partial fill logic

// estimateSplashQuote(price, amountIn) → DexQuote
//   → price * amountIn, fixed 0.3% fee assumption

// splitRoute(amountIn, routes, splitBps) → DexQuote[]
//   → TODO Phase 2: split large swaps across DEXs to minimize slippage
```

---

### Step 4 — Frontend Components

#### `SwapForm.tsx`
The main swap interface — the entire value prop:
- Token In selector + amount input (XCH or any emoji CAT)
- Token Out selector
- "↕ Flip" button to reverse the pair
- Slippage tolerance setting (0.5% / 1.0% / custom)
- Big "Find Best Rate" button → triggers mock `findBestRoute()`
- Loading state with arcane spinner

#### `RouteDisplay.tsx`
Show the best route + all alternatives:
- Winner card: Source name, amount out, price impact, fee, route path
  - e.g. `⚗️ Forge — 42.15 XCH — 0.3% impact — save 1.2% vs worst`
- Comparison table: all 4 DEX quotes side by side
  - Columns: DEX, You Get, Price Impact, Fee, Route
  - Highlight winner in gold / dim losers
- "Route saves you X XCH vs worst" banner

#### `PairSelector.tsx`
Token pair picker:
- Search by symbol or emoji (❤️ LOVE, 🌱 SPROUT, etc.)
- Recently used pairs
- All 6 emoji CATs + XCH selectable
- Shows mini price pill: `1 XCH = 42.5 LOVE`

#### `LiquidityCompare.tsx`
Side-by-side DEX comparison panel (informational):
- Table: DEX, TVL, 24h Volume, Fee Tier, Avg Slippage
- "Best for small swaps" / "Best for large swaps" badges
- Links to each DEX's own interface

---

### Step 5 — App Shell

4 tabs:
- **Swap** — main `SwapForm` + `RouteDisplay`
- **Compare** — `LiquidityCompare` across all DEXs
- **History** — past swaps (mock, stub for wallet integration)
- **Info** — how the router works, fee disclosure

Header: `⚡ Swap — swap.awizard.dev · testnet11`

---

### Step 6 — TypeScript + Build Check

```powershell
cd projects/chia-aggregator
npx tsc --noEmit   # target: 0 errors
npm run build      # target: clean dist/
```

---

## Success Criteria

| Criterion | Target |
|-----------|--------|
| `tsc --noEmit` | 0 errors |
| `npm run build` | clean dist |
| 4 DEX sources | Forge, Tibet, Dexie, Splash — all produce a mock quote |
| `findBestRoute()` | returns correct winner from 4 quotes |
| `router.ts` | pure functions, no `any`, fully typed |
| Nightspire theme | consistent with all other projects |
| Port | 5176 |
| `TabCalculated savings` | SwapForm shows "you save X XCH vs worst DEX" |

---

## Technical Notes

- **Reuse `cfmm.ts`** — copy from `chia-cfmm/src/lib/cfmm.ts` for the Forge quote calculator
- **No real HTTP calls** this phase — all quotes are computed from mock pool state constants
- **PRECISION = 1_000_000_000_000n** — same as all other projects
- **Tibet constant-product:** standard `x * y = k`, fee = 7bps (their actual testnet fee)
- **Dexie:** treat best offer as a limit order at fixed price; quote fills against it
- **Splash:** treat as oracle price × fixed 30bps fee (their quoted rate)
- **Revenue:** 5bps aggregator fee on top of DEX fee → insurance fund (stub)

---

## Parallel Safety

| Active Quest | Codebase | Overlap |
|---|---|---|
| `bootstrap-bank-of-wizards-quest` | `projects/chia-bank/` | ❌ none |
| `build-chia-multisig-interface` | new multisig project | ❌ none |
| **This quest** | `projects/chia-aggregator/` | ✅ isolated |
