# Quest: Build Wizard Analytics Hub — `stats.awizard.dev`

**Status:** ACTIVE  
**Created:** March 5, 2026  
**Champion:** aWizard 🧙  

---

## Objective

Build a **unified analytics dashboard** at `stats.awizard.dev` that aggregates metrics, volume, and activity across all wizard protocols into a single observability layer.

**The missing piece:** users have 5 DeFi apps but no way to see the big picture — total ecosystem TVL, which pools are hot, which tokens are trending, historical charts, etc.

---

## Why Now

The ecosystem is mature enough to generate meaningful data:
- ✅ **Forge** (CFMM) — swap volume, pool TVL, fee revenue
- ✅ **Craft** (emoji tokens) — token supply, price charts, market cap rankings
- ✅ **Bank** (portfolio hub) — aggregated user net worth (anonymized)
- ✅ **Treasure Chest** — listing count, total value locked in NFTs
- 🔄 **Perps** (in progress) — open interest, funding rates, liquidation events

**Zero overlap with active quests:**
- Not a DEX aggregator (bootstrap-aggregator-dex)
- Not multisig (build-chia-multisig-interface)
- Not Discord Activity UI (wire-activity-panels)
- Not code quality review (audit-code-quality)

---

## Key Features

### 📊 Ecosystem Overview
- **Total Value Locked (TVL)** — sum of all protocols in XCH + USD
- **24h Volume** — swaps across Forge pools
- **Protocol Revenue** — accumulated fees (0.3% swap + 0.05% aggregator)
- **Active Users** — unique wallet addresses interacting (7d/30d)

### 🌊 Forge (CFMM) Analytics
- **Pool Rankings** — TVL, 24h volume, APR from fees
- **Popular Pairs** — most traded pairs (XCH/LOVE, XCH/HODL, etc.)
- **Pool Detail View** — reserves chart, volume history, price chart
- **LP Leaderboard** — top liquidity providers by share

### 🪙 Emoji Token Analytics  
- **Market Cap Rankings** — LOVE, HODL, CASTER by mcap
- **Price Charts** — 7d/30d historical price vs XCH
- **Supply Analytics** — circulating supply, holder count
- **Trading Volume** — 24h volume per token

### 🎁 Treasure Chest Metrics
- **Active Listings** — count of NFTs for sale
- **Total Locked Value** — XCH + CATs locked in chests
- **Recent Sales** — last 10 purchases with price
- **Chest Activity Heatmap** — listings/sales over time

### ⚡ Perps Dashboard (when live)
- **Open Interest** — total size of all positions
- **Funding Rate** — current hourly funding for XCH-PERP
- **Liquidations** — 24h liquidation volume
- **Insurance Fund** — current balance + coverage ratio

### 🏦 Bank Insights (privacy-preserving)
- **Median Portfolio Size** — middle 50% quantile in XCH
- **Asset Distribution** — pie chart: XCH vs CATs vs LP vs NFTs (anonymized aggregate)
- **Top Held Tokens** — which emoji tokens are in most wallets

---

## Technical Architecture

### Frontend: Vite + React 19 + Recharts

**Port:** 5185 (next available after bank's 5184)

**Components:**
```
src/
├── components/
│   ├── EcosystemOverview.tsx     # Hero metrics cards
│   ├── ForgeAnalytics.tsx        # Pool tables + charts
│   ├── TokenAnalytics.tsx        # Emoji token rankings + price graphs
│   ├── ChestAnalytics.tsx        # NFT marketplace stats
│   ├── PerpsAnalytics.tsx        # Perpetuals OI + funding
│   ├── BankInsights.tsx          # Aggregated portfolio data
│   └── MetricCard.tsx            # Reusable stat display
├── lib/
│   ├── analyticsApi.ts           # Data fetching from all protocols
│   ├── chartUtils.ts             # Recharts config helpers
│   └── types.ts                  # Analytics-specific types
└── hooks/
    └── useAnalytics.ts           # React Query for data fetching
```

**Charts library:** [Recharts](https://recharts.org/) (React-native charts, works great with Radix UI)

---

### Backend: Indexer Service (Python/FastAPI)

**Location:** `projects/stats-indexer/` (new Python service)

**Responsibilities:**
1. **Poll chain state** — query Chia fullnode RPC for:
   - Pool singleton coins (CFMM reserves)
   - CAT balances (emoji token supply)
   - NFT coin states (treasure chest listings)
   - Recent spend bundles (transaction history)

2. **Cache in SQLite** — local DB with tables:
   - `pool_snapshots` (timestamp, pool_id, reserves, k_value)
   - `swap_events` (timestamp, pool_id, amount_in, amount_out, wallet)
   - `token_prices` (timestamp, asset_id, price_xch)
   - `chest_listings` (timestamp, nft_id, price, status)

3. **Expose REST API** on port **5186**:
   ```
   GET /api/tvl           → total ecosystem TVL
   GET /api/pools         → all pools with 24h stats
   GET /api/pools/:id     → single pool detail + history
   GET /api/tokens        → emoji token rankings
   GET /api/tokens/:id    → token price history
   GET /api/chests/stats  → chest marketplace metrics
   ```

4. **Update every 60s** — background task polls chain, updates cache

---

## Quest Steps

### ✅ Step 1 — Frontend Scaffold

Create `projects/chia-stats/` from template:

```powershell
robocopy projects/chia-bank projects/chia-stats /E `
  /XD node_modules dist .git `
  /XF "*.lock" /NP /NFL /NDL

# package.json: name = "chia-stats", port = 5185
# Install recharts: npm install recharts --legacy-peer-deps
# Clear src/components/* → replace with analytics components
```

### Step 2 — Mock Data Layer

Before building the real indexer, create mock API in frontend:

**`src/lib/mockAnalytics.ts`:**
```typescript
export const MOCK_TVL = 125_000_000_000_000n; // 125 XCH across all protocols

export const MOCK_POOLS = [
  {
    poolId: 'xch_love_50_50',
    symbol: 'XCH/LOVE',
    tvl: 50_000_000_000_000n,
    volume24h: 5_000_000_000_000n,
    apr: 12.5, // %
  },
  // ... more pools
];

export const MOCK_TOKENS = [
  {
    assetId: 'love_asset_id',
    symbol: 'LOVE',
    emoji: '❤️',
    mcap: 100_000_000_000_000n,
    price: 0.001, // XCH per token
    change24h: 5.2, // %
  },
  // ... more tokens
];
```

### Step 3 — Core Components

**EcosystemOverview.tsx:**
- 4 hero metric cards: TVL | 24h Volume | Revenue | Active Users
- Mini line chart showing TVL growth (7d)

**ForgeAnalytics.tsx:**
- Pool ranking table: Pool | TVL | Volume | APR | Actions → [View Details]
- Click row → expands to show reserves chart + swap history

**TokenAnalytics.tsx:**
- Token ranking cards: emoji + symbol | mcap | 24h change | price chart thumbnail
- Click → modal with full price chart (7d/30d toggle)

### Step 4 — Charts Integration

Install and configure Recharts:

```tsx
import { LineChart, Line, XAxis, YAxis, Tooltip, ResponsiveContainer } from 'recharts';

function TvlChart({ data }: { data: { timestamp: number; tvl: number }[] }) {
  return (
    <ResponsiveContainer width="100%" height={200}>
      <LineChart data={data}>
        <XAxis dataKey="timestamp" />
        <YAxis />
        <Tooltip />
        <Line type="monotone" dataKey="tvl" stroke="var(--accent)" strokeWidth={2} />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

Apply Nightspire theme to charts (custom colors, glow effects on hover).

### Step 5 — Real-time Updates

Add React Query for auto-refresh:

```tsx
import { useQuery } from '@tanstack/react-query';

function useEcosystemTvl() {
  return useQuery({
    queryKey: ['ecosystem', 'tvl'],
    queryFn: async () => {
      const res = await fetch('http://localhost:5186/api/tvl');
      return res.json();
    },
    refetchInterval: 60_000, // 1min
  });
}
```

---

## Step 6 — Indexer Service (Python)

**Location:** `projects/stats-indexer/`

**Stack:** FastAPI + SQLite + httpx (for Chia RPC)

**Files:**
```
stats-indexer/
├── main.py              # FastAPI app + endpoints
├── indexer.py           # Background poller
├── db.py                # SQLite schema + queries
├── chia_client.py       # Chia fullnode RPC wrapper
└── requirements.txt
```

**Core indexer logic:**
```python
# indexer.py
import asyncio
from chia_client import ChiaRPC

async def index_pools():
    # Query all CFMM pool singletons
    pool_coins = await chia.get_coin_records_by_puzzle_hash(POOL_PUZZLE_HASH)
    
    for coin in pool_coins:
        # Parse reserves from coin solution
        reserves = parse_pool_state(coin.coin.puzzle_reveal)
        
        # Save snapshot
        db.insert_pool_snapshot(coin.coin.name(), reserves, timestamp=now())

async def index_swaps():
    # Query recent spend bundles
    recent_tx = await chia.get_recent_spend_bundles(since=last_indexed_height)
    
    for tx in recent_tx:
        if is_swap_tx(tx):
            swap_event = parse_swap_event(tx)
            db.insert_swap_event(swap_event)
```

Deploy on same VPS as gym-server (Hetzner CPX21).

---

## Success Criteria

✅ Frontend compiles clean (`tsc --noEmit → 0`)  
✅ Dev server running on `localhost:5185`  
✅ All 6 analytics panels render with mock data  
✅ Charts display smoothly with Recharts  
✅ Matches Nightspire CSS theme (glow effects, dark mode)  
✅ Indexer service running and polling testnet11 (once contracts deployed)  
✅ Real-time data updates every 60s  

---

## Integration Points

| Protocol | Data Source | Metrics |
|----------|-------------|---------|
| **Forge (CFMM)** | Pool singleton coins | TVL, volume, APR |
| **Craft (tokens)** | CAT balance queries | Supply, price, mcap |
| **Treasure Chest** | NFT listing coins | Listings, sales, value locked |
| **Bank** | Aggregated (anonymized) | Median portfolio, asset distribution |
| **Perps** | Market singleton | OI, funding, liquidations |

---

## Timeline

**Phase 1 (this quest):** Frontend + mock data — **2-3 hours**  
**Phase 2 (future):** Python indexer + real chain data — **1-2 days**  

**Start with Phase 1** — get the UI working with beautiful mock data, then build the real indexer when contracts are deployed to testnet11.

---

*Quest active — let's build the analytics layer! 📊✨*
