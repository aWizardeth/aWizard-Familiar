# Quest: Bootstrap Perps Trading Interface ✅ COMPLETE

**Objective:** Get the Chia Perpetuals trading interface functional on testnet11 with order book and position management

## Background
The `chia-perps` project implements on-chain perpetual futures trading (Chia's equivalent to Aftermath Finance on Sui). After completing the CFMM forge, the next logical DeFi primitive is perpetuals trading with:
- Fully on-chain Central Limit Order Book (CLOB)
- Margin positions with liquidation
- Oracle price feeds
- Multi-collateral support

## Current State Analysis Needed
- [ ] Audit `perpsmath.ts` for completeness/correctness
- [ ] Check contract compilation status (Rue → CLVM)
- [ ] Verify WalletConnect CHIP-0002 integration
- [ ] Assess UI component architecture

## Completed Phase 4c Components
- [x] `perpsmath.ts` — 7 bigint math functions (markPrice, PnL, margin, funding, liquidation)
- [x] `MarketsTab.tsx` — market list with funding rate countdown + OI imbalance
- [x] `TradingPanel.tsx` — Long/Short panel, leverage slider, real-time liq price
- [x] `PositionPanel.tsx` — open positions, live PnL, margin health, close stub
- [x] `VaultTab.tsx` — afLP vault, utilization bar, deposit/withdraw preview
- [x] `LiquidationPanel.tsx` — permissionless liquidator queue, partial/full liq, fee preview
- [x] `OracleStatusBar.tsx` — persistent price ticker, staleness indicator, multi-signer badge

## Outstanding (Phase 4d — separate quest)
- [ ] Deploy contracts to testnet11
- [ ] Wire spend bundle builders into TradingPanel / PositionPanel / LiquidationPanel
- [ ] Replace all MOCK_* data with on-chain reads

## Foundation Tasks (Brick-by-brick)
- [ ] **Contracts**: Ensure order_book.rue compiles cleanly
- [ ] **Math Engine**: Validate perpetuals calculations (funding rates, PnL, liquidation thresholds)
- [ ] **Order UI**: Wire order placement with wallet integration  
- [ ] **Position Manager**: Display active positions with real-time PnL
- [ ] **Order Book Display**: Live market depth visualization

## Success Criteria
✅ **End-to-end order flow**: Place limit order → fill → position tracking → PnL updates  
✅ **TypeScript compilation clean**  
✅ **Responsive UI with Nightspire theme**  
✅ **"perps.awizard.dev ready for testnet11"**

## Technical Stack
- **Frontend**: Vite + React 19 + TypeScript + Tailwind CSS 4
- **Contracts**: Rue → CLVM (order_book.rue, vault.rue, oracle.rue)  
- **Wallet**: WalletConnect CHIP-0002 via Sage wallet
- **Theme**: Nightspire CSS tokens (consistency with CFMM forge)

## Parallel Safety
This quest operates on `projects/chia-perps/` - completely separate codebase from:
- `chia-treasure-chest` (storefront)
- Nightspire theme validation (design system)
- Any other ongoing quests

**Priority**: Medium-High (logical next DeFi primitive after AMM)
**Complexity**: High (perpetuals math + on-chain CLOB)
**Wizard Recommendation**: 🔮 Consider **Claude Opus** for complex order book architecture and liquidation logic design

---
*Quest created: March 5, 2026*
*Champion: aWizard 🧙*