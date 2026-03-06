# Quest: Enhance Bank of Wizards — Advanced Features

**Status:** BACKLOG  
**Created:** March 6, 2026  
**Champion:** aWizard 🧙  
**Priority:** Medium — Foundation complete, enhancements optional  
**Depends On:** [build-bank-of-wizards.md](../done/build-bank-of-wizards.md) ✅ COMPLETE

---

## Objective

Enhance the **Bank of Wizards** (`bank.awizard.dev`) with advanced portfolio management features:
- **Tabbed navigation** for protocol-specific positions
- **Charts & visualizations** with Recharts
- **Quick action modals** for inline swaps/sends/deposits
- **Live oracle prices** for accurate valuations
- **Testnet11 deployment** with real wallet integration

This quest handles **polish & power-user features** — the core portfolio aggregation is already complete.

---

## Foundation Status ✅

**Already delivered in Phase 1 quest:**
- ✅ Full TypeScript types (`bankTypes.ts`)
- ✅ Aggregation engine (`bankAggregator.ts`)
- ✅ Portfolio overview with net worth calculation
- ✅ Asset list (XCH + CATs + emoji tokens)
- ✅ LP position tracking with fees earned
- ✅ WalletConnect integration ready
- ✅ Nightspire theme applied
- ✅ Running on **localhost:5184**

**Current state:** 60% complete — MVP functional, enhancements needed.

---

## Enhancement Checklist

### Phase 1: Tab Structure
**Goal:** Organize positions by protocol type instead of single-page layout.

- [ ] Create `src/components/tabs/` folder
- [ ] Build `AssetsTab.tsx` — Reuse existing `AssetList.tsx`
- [ ] Build `LiquidityTab.tsx` — Reuse existing `LpPositionList.tsx`
- [ ] Build `PerpsTab.tsx` — Table of open perps positions (when perps contracts deploy)
- [ ] Build `ChestsTab.tsx` — Grid of owned Treasure Chest singletons
- [ ] Build `VaultsTab.tsx` — List of vault share balances (when vaults deploy)
- [ ] Update `App.tsx` to use Radix Tabs component
- [ ] Add tab routing with URL hash (#assets, #liquidity, etc.)

**Success:** Click tab → view filtered positions.

---

### Phase 2: Charts & Visualizations
**Goal:** Visual portfolio insights with Recharts.

#### 2a. Install Recharts
```powershell
cd projects/chia-bank
npm install recharts --legacy-peer-deps
```

#### 2b. Build Chart Components
- [ ] `PortfolioPieChart.tsx` — Asset allocation by value (XCH, CATs, LP, NFTs)
  - Use Recharts `PieChart` + `Cell` for custom colors
  - Legend shows percentage of total
  - Click slice → filter to that category
- [ ] `PerformanceChart.tsx` — Net worth over time (line chart)
  - Mock 30-day history for now (TODO: indexer service)
  - Display XCH value + USD value on dual axes
  - Tooltip shows exact value on hover
- [ ] `AssetAllocationChart.tsx` — Horizontal bar chart by protocol
  - Bars: Forge, Craft, Perps, Chests, Vaults, Wallet
  - Color-coded by Nightspire accent palette

#### 2c. Integrate Charts into Dashboard
- [ ] Add "Portfolio Breakdown" section below `PortfolioOverview`
- [ ] Place `PortfolioPieChart` on left, `AssetAllocationChart` on right
- [ ] Add toggle button: "7D / 30D / All Time" for `PerformanceChart`

**Success:** Charts render with mock data, visually appealing.

---

### Phase 3: Quick Action Modals
**Goal:** Perform common actions without leaving the Bank.

#### 3a. Build Modals
- [ ] `SwapModal.tsx` — Inline token swap
  - Token in/out selectors
  - Amount input with balance validation
  - "Get Quote" button → calls aggregator router (when deployed)
  - Display estimated output + price impact
  - "Confirm Swap" → signs + broadcasts spend bundle
- [ ] `SendModal.tsx` — Send XCH/CAT to address
  - Asset selector (XCH or any held CAT)
  - Recipient address input (bech32m validation)
  - Amount input with max button
  - Fee selector (low/medium/high)
  - "Send" → signs + broadcasts
- [ ] `DepositModal.tsx` — Add liquidity to Forge pool
  - Pool selector (from available CFMM pools)
  - Dual amount inputs for both assets
  - Preview LP tokens received
  - "Add Liquidity" → signs + broadcasts

#### 3b. Wire Up Spend Bundles
- [ ] Import `walletConnect.ts` `signCoinSpends()` + `sendTransaction()`
- [ ] Build spend bundle for swap (integrate aggregator when ready)
- [ ] Build spend bundle for send (simple XCH/CAT transfer)
- [ ] Build spend bundle for deposit (integrate CFMM logic)
- [ ] Error handling + user-friendly curse messages

#### 3c. Add Action Buttons
- [ ] Add "Swap" button to each asset row in `AssetList.tsx`
- [ ] Add "Send" button to each asset row
- [ ] Add "Manage" button to each LP position → opens Forge pool page
- [ ] Add floating action button (FAB) in bottom-right: "Quick Actions" menu

**Success:** Click "Swap LOVE" → modal opens → complete swap flow.

---

### Phase 4: Oracle Price Integration
**Goal:** Replace mock prices with live oracle feeds.

- [ ] Create `src/hooks/useOraclePrices.ts`
- [ ] Integrate **Dexie price API** (best offer book price)
  - Endpoint: `https://api.dexie.space/v1/offers` (check their docs)
  - Parse LOVE/XCH, SPROUT/XCH, etc. pair prices
- [ ] Integrate **Tibetswap price API**
  - Endpoint: TBD (check their GitHub)
  - Fetch pool reserves → calculate spot price via `x*y=k`
- [ ] Fallback to **CFMM pool prices** (our own Forge pools)
  - Use existing `cfmm.ts` `spotPrice()` function
  - Fetch pool state from indexer or WalletConnect
- [ ] Aggregate: `median(dexiePrice, tibetPrice, cfmmPrice)`
- [ ] Cache prices for 60 seconds (avoid rate limits)
- [ ] Display price staleness indicator ("Updated 15s ago")

**Success:** Asset values update with live prices every 60s.

---

### Phase 5: Testing & Polish
**Goal:** Production-ready quality.

- [ ] **Wallet Integration Testing**
  - [ ] Connect Sage wallet on testnet11
  - [ ] Verify portfolio loads real balances
  - [ ] Test swap modal with real XCH → LOVE swap
  - [ ] Test send modal with real XCH transfer
- [ ] **Loading States**
  - [ ] Add skeleton loaders for portfolio cards
  - [ ] Add spinner for chart data loading
  - [ ] Add pulse animation during price refresh
- [ ] **Error Handling**
  - [ ] Wallet disconnected → show "Connect wallet" state
  - [ ] RPC timeout → show retry button
  - [ ] Invalid address input → red border + error message
  - [ ] Insufficient balance → disable "Confirm" button
- [ ] **Responsive Design**
  - [ ] Test on mobile viewport (375px width)
  - [ ] Stack charts vertically on small screens
  - [ ] Make tabs scrollable on mobile
  - [ ] Test on tablet (768px width)
- [ ] **Accessibility**
  - [ ] Add aria-labels to buttons
  - [ ] Keyboard navigation works (Tab → Shift+Tab)
  - [ ] Focus visible on interactive elements

**Success:** All features work on live testnet11, mobile-friendly.

---

### Phase 6: Deployment
**Goal:** Deploy to production subdomain.

- [ ] Update `vercel.json` (already exists)
- [ ] Set Vercel env vars:
  - `VITE_WC_PROJECT_ID` (WalletConnect Cloud ID)
  - `VITE_NETWORK=testnet11`
- [ ] Deploy to **bank.awizard.dev**
- [ ] Test production deployment with real wallet
- [ ] Add to main aWizard site navigation
- [ ] Monitor with UptimeRobot (5-min pings)
- [ ] Community beta testing announcement

**Success:** `https://bank.awizard.dev` live on testnet11.

---

## Success Criteria

✅ **Tab navigation** — 5 tabs (Assets, Liquidity, Perps, Chests, Vaults)  
✅ **Charts render** — Pie chart, performance chart, allocation bar chart  
✅ **Quick actions work** — Swap, send, deposit modals functional  
✅ **Live prices** — Oracle integration with 60s refresh  
✅ **Testnet deployment** — bank.awizard.dev live  
✅ **Mobile responsive** — Works on phones  
✅ **Error handling** — Graceful failures with user-friendly messages

---

## Estimated Effort

**Total:** 8-12 hours  
**Breakdown:**
- Phase 1 (Tabs): 1 hour
- Phase 2 (Charts): 3 hours
- Phase 3 (Modals): 3 hours
- Phase 4 (Oracle): 2 hours
- Phase 5 (Testing): 2 hours
- Phase 6 (Deployment): 1 hour

**Priority:** Medium — Core aggregation works, these are quality-of-life improvements.

---

## Dependencies

**Blocked by:**
- Aggregator DEX deployment (for swap routing)
- Perps contracts deployment (for perps positions tab)
- Vaults deployment (for vaults tab)

**Can proceed independently:**
- Tabs, charts, send modal, oracle integration, deployment

---

**Created:** March 6, 2026  
**Foundation Complete:** build-bank-of-wizards.md ✅  
**Estimated Completion:** Q2 2026 (after aggregator + perps deploy)

---

*From good to great* 🏦✨
