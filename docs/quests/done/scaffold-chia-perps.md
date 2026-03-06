# Quest: Scaffold `chia-perps/` — The Perpetuals Exchange

> **Assigned to:** Parallel wizard (not the forge wizard)  
> **Forge wizard is:** compiling Rue contracts in `projects/chia-cfmm/contracts/`  
> **Zero overlap:** All work is in a new `projects/chia-perps/` folder — no shared files

---

## Objective

Create the `chia-perps/` project — Chia's Aftermath Finance equivalent.  
Scaffold it from `chia-cfmm/` as a template, replace the AMM-specific code with perpetuals stubs, and get it running on `localhost:5174`.

---

## Context

`chia-cfmm/` is the proven scaffold:
- Vite + React 19 + TypeScript strict
- WalletConnect CHIP-0002 via Sage wallet (`testnet11`)
- Nightspire CSS token system already in `src/App.css`
- `src/lib/walletConnect.ts` + `src/hooks/useChiaWallet.ts` (real implementation)
- Radix UI Themes v3 for component layer
- No Tailwind — Nightspire CSS vars only

`chia-perps/` inherits all of this and replaces the AMM-specific modules.

---

## Steps

### Step 1 — Copy scaffold
```powershell
robocopy .\projects\chia-cfmm .\projects\chia-perps /E /XD node_modules dist .git /XF "*.lock" /NP /NFL /NDL
```
This copies everything except `node_modules`, `dist`, `.git`, and lockfiles.

---

### Step 2 — Update `package.json`

Change:
```json
"name": "chia-cfmm"
```
To:
```json
"name": "chia-perps"
```

---

### Step 3 — Update `vite.config.ts`

Change from (no port currently set — defaults to 5173):
```ts
export default defineConfig({
  plugins: [react()],
  define: {
    global: 'globalThis',
  },
});
```
To:
```ts
export default defineConfig({
  plugins: [react()],
  define: {
    global: 'globalThis',
  },
  server: {
    port: 5174,
  },
});
```

---

### Step 4 — Replace `src/App.tsx`

Replace the entire file with a perpetuals shell:

```tsx
import './App.css';
import { Theme } from '@radix-ui/themes';
import '@radix-ui/themes/styles.css';
import { useState } from 'react';
import { useChiaWallet } from './hooks/useChiaWallet';

type Tab = 'markets' | 'trade' | 'positions' | 'vault';

export default function App() {
  const [activeTab, setActiveTab] = useState<Tab>('markets');
  const { wallet, connect, disconnect } = useChiaWallet();

  return (
    <Theme appearance="dark" accentColor="violet">
      <div style={{ minHeight: '100vh', background: 'var(--bg-deep)', color: 'var(--text-primary)' }}>

        {/* Header */}
        <header style={{
          borderBottom: '1px solid var(--border-color)',
          padding: '1rem 2rem',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'space-between',
          boxShadow: '0 0 24px var(--accent-glow)',
        }}>
          <h1 className="glow-text" style={{ margin: 0, fontSize: '1.4rem', color: 'var(--accent)' }}>
            ⚗️ Chia Perps — Testnet11
          </h1>
          <div>
            {wallet.connected ? (
              <button className="glow-btn" onClick={disconnect}>
                {wallet.address?.slice(0, 10)}…
              </button>
            ) : (
              <button className="glow-btn" onClick={connect} disabled={wallet.connecting}>
                {wallet.connecting ? 'Connecting…' : 'Connect Sage'}
              </button>
            )}
          </div>
        </header>

        {/* Tabs */}
        <nav style={{ display: 'flex', gap: '0.5rem', padding: '1rem 2rem', borderBottom: '1px solid var(--border-color)' }}>
          {(['markets', 'trade', 'positions', 'vault'] as Tab[]).map(tab => (
            <button
              key={tab}
              className={activeTab === tab ? 'glow-btn' : ''}
              onClick={() => setActiveTab(tab)}
              style={{
                padding: '0.4rem 1.2rem',
                textTransform: 'capitalize',
                opacity: activeTab === tab ? 1 : 0.5,
                cursor: 'pointer',
                background: 'transparent',
                border: activeTab === tab ? '1px solid var(--border-color)' : '1px solid transparent',
                color: 'var(--text-primary)',
                borderRadius: 6,
              }}
            >
              {tab}
            </button>
          ))}
        </nav>

        {/* Tab content placeholders */}
        <main style={{ padding: '2rem' }}>
          {activeTab === 'markets' && (
            <div className="glow-card" style={{ padding: '2rem', borderRadius: 8 }}>
              <h2>Markets</h2>
              <p style={{ opacity: 0.5 }}>// TODO: MarketsTab.tsx — list perpetual markets with mark price, 24h change, open interest</p>
            </div>
          )}
          {activeTab === 'trade' && (
            <div className="glow-card" style={{ padding: '2rem', borderRadius: 8 }}>
              <h2>Trade</h2>
              <p style={{ opacity: 0.5 }}>// TODO: TradingPanel.tsx — open long/short, size, leverage, estimated liquidation price</p>
            </div>
          )}
          {activeTab === 'positions' && (
            <div className="glow-card" style={{ padding: '2rem', borderRadius: 8 }}>
              <h2>Positions</h2>
              <p style={{ opacity: 0.5 }}>// TODO: PositionPanel.tsx — open positions: size, entry price, PnL, margin ratio, close</p>
            </div>
          )}
          {activeTab === 'vault' && (
            <div className="glow-card" style={{ padding: '2rem', borderRadius: 8 }}>
              <h2>Vault</h2>
              <p style={{ opacity: 0.5 }}>// TODO: VaultTab.tsx — afLP-equivalent vault: deposit/withdraw, TVL, APY estimate</p>
            </div>
          )}
        </main>

      </div>
    </Theme>
  );
}
```

---

### Step 5 — Replace `src/lib/types.ts`

Delete AMM types (PoolState, SwapQuote, DepositQuote etc.) and replace with perps types:

```typescript
// Perps types — mirrors on-chain structs from market_singleton.rue + account_singleton.rue

export const PRECISION = 1_000_000_000_000n; // 1e12 — matches CFMM
export const MOJOS_PER_XCH = 1_000_000_000_000n;

export interface Market {
  id: string;                  // market singleton launcher_id
  coinId: string;              // current singleton coin id
  symbol: string;              // e.g. "XCH-USDC"
  markPrice: bigint;           // PRECISION-scaled
  indexPrice: bigint;          // oracle TWAP, PRECISION-scaled
  openInterestLong: bigint;    // total long size in base mojos
  openInterestShort: bigint;
  fundingRate: bigint;         // PRECISION-scaled, positive = longs pay shorts
  nextFundingTimestamp: number;
}

export interface Position {
  id: string;                   // account singleton coin id
  marketId: string;
  isLong: boolean;
  size: bigint;                 // base asset mojos
  entryPrice: bigint;          // PRECISION-scaled
  collateral: bigint;          // XCH mojos
  unrealizedPnl?: bigint;      // computed client-side
  marginRatio?: bigint;        // computed client-side, PRECISION-scaled
  liquidationPrice?: bigint;   // computed client-side
}

export interface Order {
  id: string;
  marketId: string;
  isBid: boolean;               // true = long/buy, false = short/sell
  size: bigint;
  price: bigint;                // PRECISION-scaled limit price
  filledSize: bigint;
  timestamp: number;
}

export interface VaultState {
  totalAssets: bigint;          // XCH mojos in vault
  totalShares: bigint;          // afLP share supply
  sharePrice: bigint;           // PRECISION-scaled XCH per share
  utilizationBps: number;       // % of vault deployed as liquidity
}
```

---

### Step 6 — Create `src/lib/perpsmath.ts`

New file — all 5 stub functions, TypeScript strict, 0 errors:

```typescript
// perpsmath.ts — TypeScript reference implementation of perpetuals math
// Mirrors the on-chain logic from market_singleton.rue + liquidation_engine.rue
// All values use PRECISION = 1e12 fixed-point scaling unless noted.
//
// TODO: replace stubs with full implementations (see docs/skills/chiaPerpetuals.md)

import { PRECISION } from './types';

/** Mark price = median(bookMidPrice, fundingPrice, TWAP) */
export function markPrice(
  bookMidPrice: bigint,
  fundingPrice: bigint,
  twap: bigint,
): bigint {
  // TODO: implement median of three
  void fundingPrice; void twap;
  return bookMidPrice;
}

/** Unrealized PnL for a position at current mark price */
export function unrealizedPnl(
  size: bigint,
  entryPrice: bigint,
  currentMarkPrice: bigint,
  isLong: boolean,
): bigint {
  // TODO: (markPrice - entryPrice) * size / PRECISION for longs; inverted for shorts
  void size; void entryPrice; void currentMarkPrice; void isLong;
  return 0n;
}

/**
 * Margin ratio = collateral / (size * markPrice / PRECISION)
 * Returns PRECISION-scaled value, e.g. PRECISION / 10n = 10%
 */
export function marginRatio(
  collateral: bigint,
  size: bigint,
  currentMarkPrice: bigint,
): bigint {
  // TODO: implement
  void collateral; void size; void currentMarkPrice;
  return PRECISION;
}

/** Returns true if margin ratio < maintenanceMarginBps / 10000 */
export function isLiquidatable(
  collateral: bigint,
  size: bigint,
  currentMarkPrice: bigint,
  maintenanceMarginBps: number,
): boolean {
  // TODO: implement
  void collateral; void size; void currentMarkPrice; void maintenanceMarginBps;
  return false;
}

/**
 * Funding rate = clamp(premiumIndex, -clampBps/10000, clampBps/10000)
 * premiumIndex = (markPrice - indexPrice) / indexPrice, PRECISION-scaled
 */
export function fundingRate(
  markPriceVal: bigint,
  indexPrice: bigint,
  clampBps: number,
): bigint {
  // TODO: implement with clamp
  void markPriceVal; void indexPrice; void clampBps;
  return 0n;
}

/** Estimated liquidation price for a position */
export function liquidationPrice(
  entryPrice: bigint,
  collateral: bigint,
  size: bigint,
  maintenanceMarginBps: number,
  isLong: boolean,
): bigint {
  // TODO: entry ± (collateral - maintenanceFee) / size
  void entryPrice; void collateral; void size; void maintenanceMarginBps; void isLong;
  return 0n;
}
```

---

### Step 7 — Replace `contracts/` with perps stubs

Delete the 5 CFMM `.rue` files and create 3 new ones:

**`contracts/market_singleton.rue`**
```rust
// Market Singleton — Chia Perpetuals Exchange
// Written in Rue (https://rue-lang.com)
//
// Central singleton for a perpetual futures market.
// Holds the CLOB order book state and open interest counters.
// Processes OpenLong, OpenShort, ClosePosition, Liquidate, SettleFunding modes.
//
// TODO: implement after account_singleton.rue design is finalised.
```

**`contracts/account_singleton.rue`**
```rust
// Account Singleton — Chia Perpetuals Exchange
// Written in Rue (https://rue-lang.com)
//
// Per-user collateral vault and position map (isolated margin model).
// One singleton per user per market.
// Holds: collateral (XCH mojos), open position, entry price, funding debt.
//
// TODO: implement — design pending oracle_aggregator.rue interface.
```

**`contracts/oracle_aggregator.rue`**
```rust
// Oracle Aggregator — Chia Perpetuals Exchange
// Written in Rue (https://rue-lang.com)
//
// Aggregates price reports from 3+ oracle signers.
// Computes TWAP and publishes a price announcement that market_singleton
// and liquidation_engine consume.
//
// TODO: implement — requires BLS multi-sig design from dao_treasury.rue pattern.
```

---

### Step 8 — `npm install` and smoke test

```powershell
cd .\projects\chia-perps
npm install
npm run dev
```

Expected: Vite starts on `localhost:5174`, Nightspire dark theme renders, four tabs visible, no TypeScript errors.

---

## Definition of Done

- [x] `projects/chia-perps/` exists and `npm run dev` starts on port 5174 with 0 errors
- [x] Header: "⚗️ Chia Perps — Testnet11"
- [x] Four tabs render: Markets, Trade, Positions, Vault
- [x] Each tab shows a `glow-card` placeholder with TODO comment
- [x] Connect Sage wallet button works (real hook inherited from scaffold)
- [x] Default network: `testnet11`
- [x] `src/lib/types.ts` has Market, Position, Order, VaultState interfaces
- [x] `src/lib/perpsmath.ts` exports 7 stub functions — TypeScript strict, `tsc` 0 errors
- [x] `contracts/` has 3 `.rue` stub files (CFMM `.rue` files removed)
- [x] `package.json` name is `chia-perps`

## Do NOT touch

- `projects/chia-cfmm/` — forge wizard owns this
- `projects/chia-treasure-chest/` — separate quest
- `tests/` — test wizard owns this
- `docs/` — aWizard updates after quest is complete
