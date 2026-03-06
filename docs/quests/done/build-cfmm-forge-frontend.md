# Quest: Build CFMM Forge Frontend

**Objective:** Build the complete trading interface for the Chia CFMM (forge.awizard.dev) using the already-complete math library with mock data integration

## Background
The `chia-cfmm` project has a **complete math library** (`src/lib/cfmm.ts`) and spend bundle builders (`src/lib/spendBundles.ts`) but needs the frontend trading components. While Rue contract compilation is being resolved, we can build all UI components with mock pool data and swap them for real data later.

This creates a parallel development track separate from the ongoing `chia-perps` trading interface work.

## Current Assets Analysis ✅
- **Math Library**: Complete `cfmm.ts` with swap, liquidity, pricing calculations
- **Coin Utils**: Complete `coinUtils.ts` with Chia primitives  
- **WalletConnect**: Full CHIP-0002 integration ready
- **Theme**: Nightspire CSS tokens already integrated
- **Architecture**: Vite + React 19 + TypeScript + Tailwind CSS 4

## Foundation Tasks (Mock Data First)
- [ ] **Pool Stats Component**: `PoolStats.tsx` - display reserves, K invariant, current price, 24h volume
- [ ] **Swap Interface**: `SwapForm.tsx` - token in/out selector, amount input, price impact calculation
- [ ] **Liquidity Management**: `LiquidityForm.tsx` - add/remove liquidity, LP NFT balance display  
- [ ] **LP Position Tools**: `SplitLpModal.tsx` - split LP position interface
- [ ] **Real-time Updates**: Wire `buildSwapBundle`, `buildDepositBundle`, `buildWithdrawBundle` to components
- [ ] **Pool Discovery**: Enhanced pool selector with search/filter capabilities

## Advanced Features  
- [ ] **Slippage Protection**: Configurable slippage tolerance with warnings
- [ ] **Price Charts**: Mini candlestick chart integration for pool price history
- [ ] **Pool Analytics**: APR calculation, fee returns, impermanent loss calculator
- [ ] **Multi-hop Routing**: Route swaps through multiple pools for better rates

## Success Criteria
✅ **Complete swap flow**: Token select → amount input → preview → confirm → wallet sign  
✅ **Complete liquidity flow**: Add/remove liquidity with proper LP NFT integration  
✅ **TypeScript compilation clean**  
✅ **Responsive Nightspire-themed interface**  
✅ **"forge.awizard.dev ready for testnet11"**  

## Technical Stack  
- **Frontend**: Vite + React 19 + TypeScript + Tailwind CSS 4
- **Math**: Existing `cfmm.ts` library (Newton-Raphson, weighted AMM formulas)
- **Wallet**: WalletConnect CHIP-0002 integration via `useChiaWallet` hook  
- **Data**: Mock pool states initially, swap for real indexer later
- **Theme**: Nightspire CSS tokens with `.glow-card`, `.glow-btn` classes

## Mock Data Strategy
Create realistic pool configurations:
- **XCH/USDS Pool**: Primary stablecoin pair
- **XCH/DBX Pool**: Major CAT pair  
- **High-TVL Multi-Asset Pool**: 4-token weighted pool for advanced testing
- **Small-Cap Pool**: Low liquidity for price impact testing

## Parallel Safety
This quest operates on `projects/chia-cfmm/` frontend - completely isolated from:
- `chia-perps` trading interface (different codebase)
- Contract compilation (can use mocks)
- Backend/indexer work (frontend-first approach)
- Any world engine or other GUI work

**Priority**: High (core DeFi primitive, foundational)  
**Complexity**: Medium (leverages existing math library)  
**Wizard Recommendation**: ⚡ **SONNET comfortable** - UI scaffolding with established patterns  

---
*Quest created: March 5, 2026*  
*Champion: aWizard 🧙*  
*Parallel to: bootstrap-perps-trading-interface*