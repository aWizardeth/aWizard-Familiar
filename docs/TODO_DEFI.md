# TODO — aWizard Protocol (DeFi Ecosystem)

> Master quest log for the full aWizard DeFi build.
> Architecture: `docs/ARCHITECTURE.md` — full subdomain map + product specs.
> Projects live in `projects/` — `projects/chia-cfmm/`, `projects/chia-treasure-chest/`, `projects/chia-perps/`
>
> **Subdomain targets:** `forge.awizard.dev` (CFMM + NFT vaults), `portal.awizard.dev` (arbitrage),
> `bank.awizard.dev` (portfolio hub), `craft.awizard.dev` (emoji asset creation),
> `map.awizard.dev` (open world), `build.awizard.dev` (dev expansion), `stats.awizard.dev` (analytics)
>
> Goal: prove out the full DeFi OS on Chia testnet11, then write CHIPs for each primitive.

---

## 📊 Progress Summary

**✅ Completed Frontends (6 phases):**
- Phase 0: Theme Unification — Nightspire CSS across all projects ✅
- Phase 5: **Emoji Market** ([craft.awizard.dev](http://localhost:5183)) — 6 core tokens ✅
- Phase 6: **Bank of Wizards** ([bank.awizard.dev](http://localhost:5184)) — Portfolio aggregation MVP ✅
- Phase 7: **Analytics Hub** ([stats.awizard.dev](http://localhost:5185)) — Ecosystem metrics ✅
- Phase 10: **Vault Balancer** ([vaults.awizard.dev](http://localhost:5178)) — Foundation complete (math + scaffold) ✅

**🚧 In Progress:**
- **🚀 Deployment Quest** — Testnet11 infrastructure (Step 2/8 complete — 25%)
- Phase 1: Wallet Connect — integration ready, awaiting testnet wallet testing
- Phase 3: CFMM Math — complete, awaiting contract compilation
- Phase 4: Perpetuals — frontend complete, awaiting contracts

**📦 Ready to Deploy:**
- `chia-treasure-chest` → localhost:5180
- `chia-cfmm` → localhost:5182
- `chia-craft` → localhost:5183
- `chia-bank` → localhost:5184
- `chia-stats` → localhost:5185

**📋 Planning Backlog:**
- Future quests moved to `docs/quests/backlog/` (full Bank build, Aggregator DEX, Portal, Multisig)

---

## 🎯 Active Quests

### Phase 0 — Theme Unification (cross-project) ✅ COMPLETE
- [x] Add Nightspire CSS token block + glow utility classes to `chia-treasure-chest/src/App.css`
- [x] Add Nightspire CSS token block + glow utility classes to `chia-cfmm/src/App.css`
- [x] Update treasure-chest header / footer selectors to use `--accent`, `--border-color`, glow vars
- [x] Update CFMM header to use `--accent`, `--bg-deep`, glow shadows; `.rt-Card:hover` uses `--border-color`
- [x] Create `chia-perps/` project scaffold from `chia-cfmm/` as template
- [x] Verify all 3 sites render with `.glow-card`, `.glow-text`, `.glow-btn` classes ✅

---

### Phase 1 — Sage Wallet Connect (all frontends)

- [x] `chia-treasure-chest` — full `useChiaWallet` + `lib/walletConnect.ts` already wired (testnet11)
- [x] `chia-cfmm` — replaced mock stub with real WalletConnect Sign Client implementation
- [x] `chia-cfmm` — created `lib/walletConnect.ts` with CHIP-0002 contract methods:
  - `getAssetCoins` — query spendable CAT/NFT/DID coins
  - `getAssetBalance` — query confirmed/spendable balances
  - `signCoinSpends` — sign Rue-compiled CLVM puzzles via Sage
  - `sendTransaction` — broadcast SpendBundle to mempool
  - `signAndSend` — convenience: sign + broadcast in one call
- [x] Both projects default to `chia:testnet11` ✅
- [x] Create `chia-perps/src/` scaffold with wallet hook
- [x] Add `VITE_WC_PROJECT_ID` reminder to each `.env.example` ✅ STANDARDIZED
- [ ] Test: connect Sage wallet → address displays in header

---

### Phase 2 — Treasure Chest Frontend

Connect the existing `chia-treasure-chest` contracts to the UI.

- [ ] Read `chia-treasure-chest/contracts/` — understand deployed testnet11 singleton
- [ ] Build `ChestViewer.tsx` — displays current chest state (listings, owner, balance)
- [ ] Build `ListItemForm.tsx` — owner can list XCH, CAT, or NFT
- [ ] Build `PurchaseFlow.tsx` — buyer: view listing → confirm → submit spend bundle
- [ ] Wire spend bundle signing via `WalletConnectProvider.signCoinSpends()`
- [ ] Display Spacescan link to chest singleton coin

---

### Phase 3 — CFMM Backend Library

Implement all TypeScript back-end modules the CFMM frontend depends on.

**Math library** (`src/lib/cfmm.ts`) ✅
- [x] `powFrac(baseScaled, p, q)` — Newton-Raphson x^(p/q), 16 iterations, mirrors `pow_frac()` in swap_engine.rue
- [x] `calcSwapOut` / `calcSwapIn` — weighted CFMM with fee
- [x] `spotPrice`, `priceImpactBps`
- [x] `calcLpOut(pool, amounts)` — geometric mean (first deposit) / proportional (thereafter)
- [x] `calcWithdrawAmounts(pool, lpShare)` — proportional withdrawal
- [x] `buildSwapQuote`, `buildDepositQuote`, `buildWithdrawQuote`

**Coin utilities** (`src/lib/coinUtils.ts`) ✅
- [x] `puzzleHashToAddress` / `addressToPuzzleHash` — bech32m encode/decode (xch / txch)
- [x] `mojosToXch` / `xchToMojos`, `mojosToCat` / `catToMojos`
- [x] `coinId(parent, puzzleHash, amount)` — async SHA-256 coin ID
- [x] `bigintToChiaBytes` — Chia minimal-length big-endian integer encoding
- [x] `treeHashCons` / `treeHashAtom` — CLVM tree hash primitives
- [x] `clvmAtom`, `clvmCons`, `clvmList`, `clvmInt`, `clvmHash` — CLVM serialization

**Spend bundle builders** (`src/lib/spendBundles.ts`) ✅
- [x] `buildSwapBundle` — pool singleton Swap mode + input CAT coin spend
- [x] `buildMultiAssetDepositBundle` — MultiAssetDeposit mode + N CAT coin spends
- [x] `buildSingleAssetDepositBundle` — SingleAssetDeposit mode + 1 CAT coin spend
- [x] `buildWithdrawBundle` — Withdraw mode + LP NFT Burn mode
- [x] `buildSplitLpBundle` — LP NFT Split mode
- [x] CLVM solution serialization for all Rue struct modes (tagged-union encoding)
- [x] `PUZZLE_REVEALS` registry with clear "compile first" error messages

**Pending (connect frontend)** — ⚠️ **BLOCKED:** Rue contracts need compilation fixes
- [ ] **Compile Rue contracts** — `rue build contracts/pool_singleton.rue` + `lp_nft.rue`; fill `PUZZLE_REVEALS` in spendBundles.ts
- [ ] Build `PoolStats.tsx` — reserves, K invariant, current price, 24h volume
- [ ] Build `SwapForm.tsx` — token in/out with estimated output and price impact; wire `buildSwapBundle`
- [ ] Build `LiquidityForm.tsx` — add/remove liquidity, show LP NFT balance; wire deposit/withdraw bundles
- [ ] Build `SplitLpModal.tsx` — split LP position; wire `buildSplitLpBundle`
- [ ] Real `poolIndexer.ts` — replace mock with WalletConnect getPublicKeys → pool coin fetch; populate `currentPuzzleReveal` + `lineageProof`
- [ ] Display pool singleton coin on Spacescan testnet11

---

### Phase 4 — Chia Perpetuals (`chia-perps`)

Build the Aftermath-equivalent perpetuals exchange on Chia. Start from testnet11.

#### 4a. Contracts (Rue/CLVM)
- [ ] Design `market_singleton.rue` — CLOB state, open interest, funding accumulator
- [ ] Design `account_singleton.rue` — per-user collateral + position map (isolated margin)
- [ ] Design `oracle_aggregator.rue` — multi-source price aggregation (3+ signed oracle coins)
- [ ] Design `insurance_fund.rue` — market-isolated insurance pool
- [ ] Design `liquidation_engine.rue` — permissionless liquidation logic
- [ ] Design `aflp_vault.rue` — community LP market-making vault

#### 4b. Math (TypeScript reference) ✅ COMPLETE
- [x] `src/lib/perpsmath.ts` — mark price median: `median(bookPrice, fundingPrice, TWAP)`
- [x] `src/lib/perpsmath.ts` — unrealized PnL: `(markPrice - entryPrice) × size`
- [x] `src/lib/perpsmath.ts` — margin ratio check for liquidation trigger
- [x] `src/lib/perpsmath.ts` — partial liquidation: minimum size to restore health
- [x] `src/lib/perpsmath.ts` — funding rate computation (premium × clamp)

#### 4c. Frontend Components
- [x] `MarketsTab.tsx` — ✅ COMPLETE list of perpetual markets with mark price, 24h change, OI
- [x] `TradingPanel.tsx` — open long/short, size, leverage input, estimated liquidation price
- [x] `PositionPanel.tsx` — open positions: size, entry price, PnL, margin ratio, close button
- [x] `LiquidationPanel.tsx` — permissionless liquidator view: underwater positions + liquidate btn
- [x] `VaultTab.tsx` — afLP-equivalent vault: deposit/withdraw, TVL, APY estimate
- [x] `OracleStatusBar.tsx` — show current oracle prices + staleness indicator

#### 4d. Testnet Deployment
- [ ] Deploy `oracle_aggregator` singleton on testnet11
- [ ] Deploy `market_singleton` for XCH-PERP on testnet11
- [ ] Deploy `insurance_fund` singleton
- [ ] Open test positions via UI, verify on Spacescan
- [ ] Run a test liquidation (open undercollateralised position, trigger liquidator)
- [ ] Document coin IDs for CHIP submission evidence

---

### Phase 5 — Emoji Market (craft.awizard.dev) ✅ COMPLETE

Create and launch the 6 core emoji token CATs on testnet11 via `craft.awizard.dev`.

- [ ] Design `emoji_cat_launcher.rue` — standard CAT launcher with emoji metadata
- [ ] Mint 6 genesis emoji tokens on testnet11:
  - [ ] ❤️  LOVE — Love / social capital
  - [ ] 🌱 SPROUT — Growth / ecological
  - [ ] 🔮 CASTER — Magic / creative power
  - [ ] ✨ SPELL — Utility / action fuel
  - [ ] ⚡ POWER — Energy / compute
  - [ ] 💎 HODL — Store of value / conviction
- [ ] Create Forge pools: LOVE/XCH, SPROUT/XCH, CASTER/XCH, SPELL/XCH, POWER/XCH, HODL/XCH
- [x] Build `craft.awizard.dev` UI ✅ **COMPLETE**
  - [x] `EmojiTokenGrid.tsx` — 6 token cards with emoji, symbol, rarity glow
  - [x] `TokenCreator.tsx` — minting interface with cost calculation
  - [x] `TokenMetadata.tsx` — full token details with backstory + use cases
  - [x] Core token definitions with rich metadata (emotion, backstory, useCase)
  - [x] Dev server running on **localhost:5183**
- [ ] Add emoji token metadata standard to CHIP submission queue

---

### Phase 6 — Bank of Wizards (bank.awizard.dev) ✅ MVP COMPLETE

**Quest:** [docs/quests/done/build-bank-of-wizards.md](quests/done/build-bank-of-wizards.md) ✅  
**Enhancements:** [docs/quests/backlog/enhance-bank-of-wizards.md](quests/backlog/enhance-bank-of-wizards.md) (backlog)  
**Status:** Foundation delivered (60%), enhancements deferred

Portfolio hub — aggregates everything a user owns across all wizard protocols.

**✅ Delivered (MVP Foundation):**
- [x] Scaffold `projects/chia-bank/` from `chia-cfmm` template ✅
- [x] `PortfolioOverview.tsx` — total net worth in XCH + USD, breakdown by category ✅
- [x] `AssetList.tsx` — all CAT/emoji tokens with balance, price, value ✅
- [x] `LpPositionList.tsx` — LP NFT positions with share %, fees earned, manage links ✅
- [x] Core types: `AssetBalance`, `LpPosition`, `TreasureChest`, `EmojiTokenBalance` ✅
- [x] `bankAggregator.ts` — net worth calculation, price formatting ✅
- [x] Mock data integration (ready for real WalletConnect queries) ✅
- [x] Dev server running on **localhost:5184** ✅

**📦 Deferred to Backlog (Enhancements):**
- Tab structure (Assets, Liquidity, Perps, Chests, Vaults)
- Charts & visualizations (Recharts pie/line/bar charts)
- Quick action modals (swap, send, deposit)
- Oracle price integration (Dexie, Tibetswap, CFMM fallback)
- Testnet11 deployment to bank.awizard.dev

---

### Phase 7 — Analytics Hub (stats.awizard.dev) ✅ COMPLETE

Unified analytics dashboard — ecosystem observability layer with metrics, charts, and rankings.

- [x] Scaffold `projects/chia-stats/` from `chia-bank` template ✅
- [x] Install Recharts charting library ✅
- [x] Mock analytics data layer with realistic ecosystem metrics ✅
- [x] `EcosystemOverview.tsx` ✅
  - [x] Hero TVL card (125.75 XCH total ecosystem)
  - [x] 7-day TVL growth chart with Recharts
  - [x] Metric cards: 24h volume, protocol revenue, active users, protocol count
- [x] `ForgeAnalytics.tsx` — CFMM pool rankings table ✅
  - [x] Pool list sorted by TVL
  - [x] Display: weights, TVL, 24h volume, APR
  - [x] 6 pools (XCH/LOVE leads with 45 XCH + 14.2% APR)
- [x] `TokenAnalytics.tsx` — emoji token market cap leaderboard ✅
  - [x] Token cards with rank, emoji, mcap, 24h change
  - [x] Price, supply, holder count stats
  - [x] LOVE #1 (100 XCH mcap), HODL #2 (80 XCH mcap)
- [x] Recharts integration with Nightspire theme (accent colors, glow effects) ✅
- [x] Dev server running on **localhost:5185** ✅
- [ ] Build Python indexer service (FastAPI + SQLite)
- [ ] Poll chain state every 60s (pool coins, CAT balances, NFT listings)
- [ ] REST API for real-time data (`/api/tvl`, `/api/pools`, `/api/tokens`)
- [ ] Replace mock data with live testnet11 queries

---

### 🚀 Deployment Quest — Testnet11 Infrastructure ⚡ IN PROGRESS

**Quest Doc:** [docs/quests/deploy-testnet-infrastructure.md](quests/deploy-testnet-infrastructure.md)  
**Deployment Map:** [docs/DEPLOYMENT_MAP.md](DEPLOYMENT_MAP.md)  
**Setup Guides:** [WALLETCONNECT_SETUP.md](WALLETCONNECT_SETUP.md) | [VERCEL_SETUP.md](VERCEL_SETUP.md) | [RAILWAY_SETUP.md](RAILWAY_SETUP.md) | [ENV_VARS_REFERENCE.md](ENV_VARS_REFERENCE.md)

Deploy all 11 projects to testnet11 with production CI/CD:

- [x] **Step 1: Domain & Hosting Setup** ✅ COMPLETE
  - [x] Inventory all projects → 11 total (9 Vite frontends + gym-server + bow-app)
  - [x] Map subdomains: forge/craft/bank/chest/perps/vaults/stats/faucet/map/gym.awizard.dev
  - [x] Create `vercel.json` for all 8 Vite frontends
  - [x] Document `.env.example` files (production-ready)
  - [x] Create WalletConnect Cloud setup guide
- [x] **Step 2: Environment Configuration** ✅ COMPLETE
  - [x] Create **WALLETCONNECT_SETUP.md** — 7-step guide to WalletConnect Cloud project setup
  - [x] Create **VERCEL_SETUP.md** — Complete Vercel config for all 8 frontends
  - [x] Create **RAILWAY_SETUP.md** — gym-server deployment with persistent SQLite
  - [x] Create **ENV_VARS_REFERENCE.md** — Master reference for 37 environment variables
- [ ] **Step 3: CI/CD Pipeline Setup** *(backlog — do after local testing validates the UI)*
  - [x] `chia-cfmm` production-ready: SPA rewrites, chunk splitting, forge.awizard.dev metadata ✅ **2026-03-06**
  - [ ] **Deploy forge.awizard.dev** — connect Vercel project to `projects/chia-cfmm/`, set `VITE_WC_PROJECT_ID` + `VITE_CHIA_NETWORK=testnet11` env vars
  - [ ] Install Vercel GitHub App → link remaining 7 frontend projects
  - [ ] Configure Railway auto-deploy for gym-server
  - [ ] Enable preview deployments for PR branches
  - [ ] Test deployment flow (push to main → auto-deploy)
- [ ] **Step 4: Testnet11 Wallet Infrastructure**
  - [ ] Generate deployment wallets (1 per contract project)
  - [ ] Fund via testnet11 faucet (1000 XCH each)
  - [ ] Document fingerprints + mnemonics in 1Password
  - [ ] Deploy gym-server wallet for battle signing
- [ ] **Step 5: Monitoring & Logging**
  - [ ] Enable Vercel Analytics (all frontends)
  - [ ] Configure Sentry error tracking
  - [ ] Set up UptimeRobot monitoring (5-min pings)
  - [ ] Railway metrics dashboard (gym-server CPU/memory)
- [ ] **Step 6: Deployment Documentation**
  - [ ] Write DEPLOYMENT.md runbook
  - [ ] Update project READMEs with production URLs
  - [ ] Create troubleshooting guide
- [ ] **Step 7: Testing & Validation**
  - [ ] Smoke test all frontends (wallet connect, UI loads)
  - [ ] Backend health checks (gym-server /health endpoint)
  - [ ] Cross-service integration tests
  - [ ] Lighthouse benchmarks (performance audit)
- [ ] **Step 8: Production Readiness Checklist**
  - [ ] HTTPS verification (all subdomains)
  - [ ] CORS configuration (gym-server → map.awizard.dev)
  - [ ] Error boundaries on all React roots
  - [ ] Testnet disclaimers on every frontend

---

### Phase 8 — Aggregator DEX (swap.awizard.dev)

Unified swap interface — aggregates liquidity from all major Chia DEXs.

- [ ] Scaffold `projects/chia-aggregator/` from `chia-cfmm` template
- [ ] **Dexie Integration:** Parse offer book API, calculate swap routes via orderbook matching
- [ ] **Tibetswap Integration:** Import pool configs, calculate AMM swap rates
- [ ] **Splash Integration:** Fetch pool state, calculate swap quotes via their SDK
- [ ] **Our CFMM Integration:** Use existing `cfmm.ts` math library with fetched pool state
- [ ] **Smart Router:** `findBestRoute(tokenIn, tokenOut, amountIn)` — compares rates across all DEXs
- [ ] **Atomic Splitting:** Route large swaps across multiple DEXs to minimize slippage
- [ ] `AggregatorSwapForm.tsx` — unified UI showing best rate + selected route
- [ ] `LiquidityCompare.tsx` — side-by-side TVL, APR, volume comparison across DEXs
- [ ] **Revenue Sharing:** 0.05% protocol fee on aggregated swaps → insurance fund
- [ ] Deploy on `swap.awizard.dev` with unified brand

### Phase 9 — Portal Arbitrage (portal.awizard.dev)

Market balancer — detects price mismatches, routes arbitrage bundles.

- [ ] `priceMonitor.ts` — poll Forge pool prices + Dexie offer book prices
- [ ] `arbDetector.ts` — compute profitable arbitrage routes
- [ ] `buildArbBundle()` — construct atomic spend bundle (swap Forge → capture spread)
- [ ] `portal.awizard.dev` UI — display active opportunities, historical captures
- [ ] Feed arb revenue to insurance fund singleton

---

### Phase 10 — Vault Balancer (Auto-Rebalancing Vaults) ✅ FOUNDATION COMPLETE

**Quest:** [docs/quests/done/build-vault-balancer-system.md](quests/done/build-vault-balancer-system.md) ✅  
**Enhancements:** [docs/quests/backlog/enhance-vault-balancer-system.md](quests/backlog/enhance-vault-balancer-system.md) (backlog)  
**Project:** [projects/chia-vaults/](../projects/chia-vaults/)  
**Status:** Foundation delivered (30%), contracts + frontend deferred

Automated liquidity management vaults for CFMM pools and Treasure Chests — Chia's equivalent
of Aftermath afLP vaults. Community-owned singletons that provide passive market making with
automated rebalancing based on oracle prices.

**✅ Delivered (Foundation Complete):**
- [x] Quest specification with full Aftermath afLP reference (900+ lines) ✅
- [x] Contract flow diagrams (Mermaid: deposit, withdraw, rebalance, oracle) ✅
- [x] Math library: `src/lib/vaultBalancer.ts` ✅ COMPLETE (18 KB, 600+ lines)
  - Share calculations (deposit/withdraw)
  - Rebalance trigger detection & optimal action computation
  - APY tracking, impermanent loss calculations
  - Oracle price aggregation (median of 3+ sources)
  - Risk metrics (VaR, Sharpe ratio)
- [x] Project scaffold: `projects/chia-vaults/` fully scaffolded ✅
- [x] README.md with comprehensive documentation ✅
- [x] Dev server ready on **localhost:5178** ✅

**📦 Deferred to Backlog (70% remaining):**
- Rue smart contracts (`aflp_vault.rue`, `chest_vault.rue`, `vault_share.rue`)
- Frontend UI (VaultDashboard, DepositFlow, WithdrawFlow, RebalanceMonitor)
- Spend bundle builders for contract interactions
- Oracle integration (Dexie, Tibetswap, CFMM price feeds)
- Vault indexer for live state queries
- Testnet deployments (CFMM vault + Chest vault)
- CHIP documentation & submission

---

### Phase 10 — CHIP Submissions

After testnet proofs are live:

- [ ] Treasure Chest CHIP — NFT programmable vault standard (Informational)
- [ ] CFMM CHIP — weighted multi-CAT AMM primitive (Standards Track)
- [ ] Emoji Token CHIP — emoji metadata standard for CATs
- [ ] Perps CHIP-A: On-Chain CLOB Standard
- [ ] Perps CHIP-B: Perpetual Futures Position Standard
- [ ] Perps CHIP-C: Oracle Aggregation Standard
- [ ] Perps CHIP-D: Permissionless Vault Standard
- [ ] Vault Balancer CHIP — Auto-Rebalancing Liquidity Vault Standard (Informational)

---

## 🏁 Completed

- ✅ **2026-03-05** — `docs/skills/chiaPerpetuals.md` created (full Aftermath-equivalent protocol spec)
- ✅ **2026-03-05** — `docs/skills/nightspireTheme.md` created (canonical Nightspire CSS design system)
- ✅ **2026-03-05** — `awizard.agent.md` updated with Chia DeFi ecosystem, Nightspire theme convention, skills table
- ✅ **2026-03-05** — Phase 0: Nightspire glow tokens + utility classes added to both `App.css` files ✅ THEME VALIDATION COMPLETE
- ✅ **2026-03-05** — Phase 1: `chia-cfmm/lib/walletConnect.ts` created with full CHIP-0002 contract methods ✅ WIRE CFMM COMPLETE  
- ✅ **2026-03-05** — Phase 1: `chia-cfmm/hooks/useChiaWallet.ts` rewritten (mock → real WalletConnect) 
- ✅ **2026-03-05** — Phase 4b: `chia-perps/src/lib/perpsmath.ts` complete with all perpetuals math functions ✅ PERPS MATH COMPLETE
- ✅ **2026-03-05** — Phase 4c: `chia-perps/MarketsTab.tsx` built with markets display ✅ MARKETS TAB COMPLETE
- ✅ **2026-03-05** — World Engine: `awizard-gui/` Phaser 3 tile renderer + world foundation ✅ BOOTSTRAP WORLD COMPLETE
- ✅ **2026-03-05** — Sage CHIP-0002 method list documented in `/memories/chia-ecosystem.md`
- ✅ **2026-03-05** — Phase 10: Vault Balancer Foundation (Phases 1-2) ✅ MATH LIBRARY + QUEST DOC COMPLETE
  - Quest: `build-vault-balancer-system.md` (900+ lines, contract flows, full spec)
  - Math library: `chia-vaults/src/lib/vaultBalancer.ts` (600+ lines, all vault math)
  - Project scaffold: `chia-vaults/` ready for contract development
- ✅ **2026-03-06** — Phase 6: Bank of Wizards MVP ✅ FOUNDATION COMPLETE
  - Quest: [build-bank-of-wizards.md](quests/done/build-bank-of-wizards.md) moved to done/
  - Core aggregation engine (60% complete) — net worth, assets, LP positions
  - Enhancements backlogged: [enhance-bank-of-wizards.md](quests/backlog/enhance-bank-of-wizards.md)
- ✅ **2026-03-06** — Phase 10: Vault Balancer Foundation ✅ COMPLETE (30%)
  - Quest: [build-vault-balancer-system.md](quests/done/build-vault-balancer-system.md) moved to done/
  - Math library complete (`vaultBalancer.ts` — 18 KB with all vault math)
  - Project scaffold ready (`projects/chia-vaults/` on localhost:5178)
  - Enhancements backlogged: [enhance-vault-balancer-system.md](quests/backlog/enhance-vault-balancer-system.md)

---

## 💡 Ideas Parking Lot

- **Cross-collateral margin**: use CFMM LP NFTs as collateral in the perps account singleton
- **Insurance fund mining**: emit reward CATs to liquidators proportional to bad debt covered
- **Referral codes**: builder fee split (like Aftermath's builder codes) — route % to a referrer coin
- **Multi-vault strategies**: vault-of-vaults that allocates to multiple afLP vaults based on APY
- **Yield tokenization**: split vault shares into principal + yield tokens (Pendle-style)

---

_Last updated: 2026-03-06_
