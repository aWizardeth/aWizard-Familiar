# Quest: Enhance Vault Balancer System — Contracts & Frontend

**Status:** BACKLOG  
**Created:** March 6, 2026  
**Champion:** aWizard 🧙  
**Priority:** High — Critical DeFi primitive (Phase 10 from roadmap)  
**Depends On:** [build-vault-balancer-system.md](../done/build-vault-balancer-system.md) ✅ COMPLETE

---

## Objective

Complete the **Vault Balancer System** by implementing:
- **Rue smart contracts** for afLP vault logic
- **Frontend UI** for vault management
- **Spend bundle builders** for contract interactions
- **Oracle integration** for rebalance triggers
- **Testnet deployment** for both CFMM and Treasure Chest vaults
- **CHIP documentation** for standards submission

This quest handles **contract development & deployment** — the foundation (research, math, scaffold) is already complete.

---

## Foundation Status ✅

**Already delivered in Phase 1 quest:**
- ✅ Full research document (Aftermath afLP mechanics)
- ✅ Contract flow diagrams (Mermaid: deposit, withdraw, rebalance, oracle)
- ✅ Complete math library (`vaultBalancer.ts` — 18 KB, 600+ lines)
  - Share calculations, rebalance triggers, APY tracking
  - Impermanent loss calculations, oracle aggregation
  - Risk metrics (VaR, Sharpe ratio)
- ✅ Project scaffold (`projects/chia-vaults/`)
- ✅ README.md comprehensive documentation
- ✅ Running on **localhost:5178** (scaffold ready)

**Current state:** 30% complete — Math + research done, contracts + UI needed.

---

## Enhancement Checklist

### Phase 3: Rue Smart Contracts
**Goal:** Implement on-chain vault logic.

#### 3a. afLP Vault Contract (`contracts/aflp_vault.rue`)
- [ ] **Deposit Mode** — Accept XCH/CAT, mint vault shares
  - Calculate share amount via geometric mean (first deposit) or proportional (subsequent)
  - Update total shares, reserve amounts
  - Emit vault share CAT tokens
- [ ] **Withdraw Mode** — Burn shares, return proportional assets
  - Validate share ownership
  - Calculate proportional amounts for each reserve asset
  - Apply withdrawal fee (0.1% to prevent flash attacks)
  - Destroy share tokens
- [ ] **Rebalance Mode** — Adjust pool weights based on oracle prices
  - Verify oracle proof signatures
  - Check drift exceeds threshold (5% default)
  - Compute optimal rebalance action (which assets to add/remove)
  - Execute swap against CFMM pool
  - Collect rebalancer fee (0.1% to caller)
- [ ] **Compound Mode** — Reinvest accumulated fees into vault
  - Calculate total fees earned since last compound
  - Swap fees for optimal asset distribution
  - Add liquidity to underlying pools
  - Increase share value (not share count)
- [ ] **Oracle Price Update** — Update cached oracle price for rebalance checks
  - Aggregate 3+ oracle signatures
  - Validate median price
  - Update last oracle timestamp

#### 3b. Vault Share Token (`contracts/vault_share.rue`)
- [ ] Standard CAT implementation
- [ ] Metadata: vault ID, share symbol (e.g., "vLOVE-XCH")
- [ ] Transferrable shares (allow secondary market)

#### 3c. Treasure Chest Vault (`contracts/chest_vault.rue`)
- [ ] Specialized vault for NFT pricing automation
- [ ] Floor oracle integration (read NFT floor price)
- [ ] Auto-listing strategy (list at floor + premium)
- [ ] Stop-loss triggers (remove listing if floor crashes)
- [ ] Take-profit triggers (remove if floor spikes)

#### 3d. Compile & Test
- [ ] Compile all contracts to CLVM
- [ ] Unit test each spend mode (Python brun simulations)
- [ ] Fuzz test rebalance logic with random oracle prices
- [ ] Security audit checklist (reentrancy, oracle manipulation, fee drains)

**Success:** All contracts compile cleanly, pass unit tests.

---

### Phase 4: Frontend Components
**Goal:** Build vault management UI.

#### 4a. Core Components
- [ ] `VaultDashboard.tsx` — Overview of all vaults
  - Grid of vault cards: name, TVL, APY, 24h change
  - Filter by vault type (CFMM, Chest)
  - Sort by TVL, APY, creation date
- [ ] `VaultDetailPage.tsx` — Single vault deep dive
  - Hero stats: TVL, APY, your shares, your value
  - Reserve breakdown chart (pie chart)
  - Rebalance history timeline
  - Oracle price vs. pool price comparison
- [ ] `VaultList.tsx` — Table of available vaults
  - Columns: Name, Type, TVL, APY, Your Shares, Actions
  - Click row → navigates to VaultDetailPage

#### 4b. Deposit/Withdraw Flows
- [ ] `DepositFlow.tsx` — Deposit assets into vault
  - Select vault from dropdown
  - Choose deposit assets (single or multi-asset)
  - Amount input with balance validation
  - Preview: shares received, estimated APY
  - "Deposit" button → signs spend bundle via WalletConnect
- [ ] `WithdrawFlow.tsx` — Redeem shares for assets
  - Select vault
  - Enter shares to burn (or "Max" button)
  - Preview: assets received (proportional to reserves)
  - Withdrawal fee display (0.1%)
  - "Withdraw" button → signs spend bundle

#### 4c. Rebalancing & Oracle
- [ ] `RebalanceMonitor.tsx` — Real-time rebalance status
  - Oracle price vs. pool price drift meter
  - "Rebalance Needed" indicator when drift > threshold
  - "Trigger Rebalance" button (permissionless)
  - Estimated rebalancer fee (0.1% reward)
  - Last rebalance timestamp
- [ ] `PerformanceChart.tsx` — Vault APY over time
  - Line chart with Recharts
  - Comparison: vault APY vs. passive hold APY
  - Impermanent loss overlay
  - Date range selector (7D, 30D, All Time)
- [ ] `ImpermanentLossCalculator.tsx` — Educational tool
  - Inputs: initial deposit amounts, current prices
  - Outputs: IL percentage, IL in XCH, vault strategy savings

#### 4d. Treasure Chest Vault UI
- [ ] `ChestVaultManager.tsx` — Manage chest vault settings
  - Floor oracle display (current NFT floor price)
  - Premium setting (e.g., "List at floor + 20%")
  - Stop-loss threshold (e.g., "Remove if floor < 0.5 XCH")
  - Take-profit threshold (e.g., "Remove if floor > 2 XCH")
  - Auto-listing toggle (on/off)
  - Current listings table with status

**Success:** All UI components render, deposit/withdraw flows work with mock data.

---

### Phase 5: Spend Bundle Builders
**Goal:** Wire frontend to contract interactions.

- [ ] `buildDepositBundle()` — Construct deposit spend bundle
  - User CAT/XCH coin spends
  - Vault singleton spend (Deposit mode)
  - Vault share CAT minting
  - Proper announcements for indexer
- [ ] `buildWithdrawBundle()` — Construct withdrawal spend bundle
  - Vault share CAT burn
  - Vault singleton spend (Withdraw mode)
  - Proportional asset output coins
  - Withdrawal fee handling
- [ ] `buildRebalanceBundle()` — Construct rebalance spend bundle
  - Oracle proof aggregation (3+ signatures)
  - Vault singleton spend (Rebalance mode)
  - CFMM pool swap execution
  - Rebalancer fee output
- [ ] `buildCompoundBundle()` — Auto-compound accumulated fees
  - Vault singleton spend (Compound mode)
  - Fee collection calculation
  - Optimal swap routing
  - LP position adjustment
- [ ] `buildChestPriceUpdateBundle()` — Update chest vault oracle
  - NFT floor oracle proof
  - Chest vault singleton spend (Oracle Update mode)
  - Listing price recalculation
  - Auto-listing logic execution

**Success:** All bundles sign and broadcast successfully on testnet11.

---

### Phase 6: Oracle Integration
**Goal:** Wire live price feeds for rebalance triggers.

- [ ] Design OracleProof format (BLS signatures + price data)
- [ ] Integrate **Dexie TWAP** price feed
  - API endpoint: TBD (check Dexie docs)
  - Parse price history → calculate TWAP
  - Cache for 60 seconds
- [ ] Integrate **Tibetswap** pool prices
  - Fetch pool reserves via API
  - Calculate spot price via `x*y=k`
  - Cache for 60 seconds
- [ ] Integrate **Forge CFMM** pool prices (fallback)
  - Use existing `cfmm.ts` `spotPrice()` function
  - Fetch pool state from indexer or WalletConnect
- [ ] Implement `aggregateOraclePrices()` frontend
  - Collect 3+ price sources
  - Compute median price
  - Display staleness indicator ("Updated 15s ago")
  - Sign oracle proof with mock BLS signer (testnet only)
- [ ] Mock oracle server for testing
  - FastAPI endpoint returning signed price proofs
  - Random price variations for rebalance testing

**Success:** Rebalance trigger activates when oracle price drifts > 5%.

---

### Phase 7: Vault Indexer
**Goal:** Query vault state from blockchain.

- [ ] Implement `fetchVaultState()` — Read vault singleton from chain
  - Query vault coin via WalletConnect or full node RPC
  - Parse CLVM solution to extract reserves, shares, last rebalance
  - Return VaultState type
- [ ] Implement `fetchUserShares()` — Read user's vault share balance
  - Query vault share CAT coins owned by user wallet
  - Sum total shares
  - Calculate % of vault ownership
- [ ] Implement `fetchVaultHistory()` — Historical rebalance events
  - Parse on-chain announcements for Rebalance mode spends
  - Extract timestamp, assets swapped, oracle price
  - Build rebalance timeline
- [ ] WebSocket subscription for real-time updates
  - Subscribe to vault singleton coin updates
  - Emit event when vault state changes
  - Auto-refresh UI on deposit/withdraw/rebalance

**Success:** VaultDetailPage loads live data from testnet11 vault.

---

### Phase 8: Testnet Deployment (CFMM Vault)
**Goal:** Deploy first afLP vault on testnet11.

- [ ] Deploy `aflp_vault` singleton for **XCH/LOVE pool**
  - Initial reserves: 10 XCH + 1000 LOVE (target 50/50 weight)
  - Performance fee: 2%
  - Rebalance threshold: 5%
- [ ] Deploy `vault_share` CAT (symbol: "vLOVE-XCH")
- [ ] Seed initial liquidity (deposit 10 XCH from test wallet)
- [ ] Perform test deposit (add 5 XCH, verify shares minted)
- [ ] Manipulate oracle price to trigger rebalance
  - Set oracle: 1 LOVE = 0.015 XCH (50% drift)
  - Call `buildRebalanceBundle()`
  - Verify vault rebalances pool weights
- [ ] Perform test withdrawal (burn 50% shares, verify assets returned)
- [ ] Document Spacescan coin IDs:
  - Vault singleton: `0xVAULT_ID`
  - Share CAT tail: `0xSHARE_TAIL`
  - Example rebalance transaction

**Success:** Full deposit → rebalance → withdraw cycle works on testnet11.

---

### Phase 9: Testnet Deployment (Chest Vault)
**Goal:** Deploy NFT pricing vault.

- [ ] Create test NFT collection (Wizard Hat NFTs, 10 copies)
- [ ] Create Treasure Chest with 1 Wizard Hat listed
- [ ] Deploy `chest_vault` singleton linked to chest
- [ ] Set floor oracle: 1 Wizard Hat = 0.5 XCH
- [ ] Set auto-listing premium: floor + 20% = 0.6 XCH
- [ ] Trigger oracle price update (floor drops to 0.3 XCH)
  - Verify vault updates listing to 0.36 XCH
- [ ] Trigger stop-loss (floor drops to 0.1 XCH, < threshold)
  - Verify vault removes listing
- [ ] Document Spacescan coin IDs

**Success:** Chest vault dynamically adjusts NFT listing price based on oracle.

---

### Phase 10: Documentation & CHIP Submission
**Goal:** Publish standards documentation.

- [ ] Write `ARCHITECTURE.md` for chia-vaults project
  - Vault lifecycle diagrams
  - Rebalancing algorithm explanation
  - Oracle aggregation security model
- [ ] Write `REBALANCING.md` — Deep dive on rebalance logic
  - Math formulas with examples
  - Threshold tuning recommendations
  - Gas cost analysis
- [ ] Write `ORACLE_INTEGRATION.md` — Oracle design
  - BLS signature verification
  - Median aggregation rationale
  - Attack vectors & mitigations
- [ ] Add inline contract comments (Rue files)
  - Every spend mode documented
  - Security invariants explained
- [ ] Draft **CHIP-XXX: Auto-Rebalancing Liquidity Vault Standard**
  - Type: Informational (or Standards Track if singleton pattern reusable)
  - Abstract, Motivation, Specification, Security, Rationale
  - Reference implementation: link to chia-vaults repo
- [ ] Submit CHIP to GitHub for community review
- [ ] Present at Chia Developer Forum

**Success:** CHIP accepted, vault balancer recognized as ecosystem primitive.

---

## Success Criteria

✅ **Contracts deployed** — afLP vault + chest vault on testnet11  
✅ **Full UI functional** — Deposit, withdraw, rebalance flows work  
✅ **Oracle-driven rebalancing** — Automatic rebalance when drift > 5%  
✅ **Testnet evidence** — Spacescan links to all vault transactions  
✅ **CHIP drafted** — Community feedback received  
✅ **Production ready** — vaults.awizard.dev deployed

---

## Estimated Effort

**Total:** 40-60 hours  
**Breakdown:**
- Phase 3 (Contracts): 16 hours
- Phase 4 (Frontend): 12 hours
- Phase 5 (Spend Bundles): 8 hours
- Phase 6 (Oracle): 6 hours
- Phase 7 (Indexer): 4 hours
- Phase 8 (CFMM Testnet): 6 hours
- Phase 9 (Chest Testnet): 4 hours
- Phase 10 (Documentation): 6 hours

**Priority:** High — Critical DeFi primitive for ecosystem bootstrapping.

---

## Dependencies

**Blocked by:**
- Rue compiler stability (contract compilation issues)
- Oracle infrastructure (need BLS signature service for testnet)
- CFMM pool liquidity (need sufficient XCH/LOVE depth for rebalance testing)

**Can proceed independently:**
- Frontend development (use mock contract responses)
- Oracle integration (mock oracle server)
- Documentation (reference implementation via TypeScript math)

---

## Technical Notes

- **Reuse CFMM math:** `cfmm.ts` from chia-cfmm for pool calculations
- **Reuse perps oracle:** Adapt oracle design from chia-perps (median aggregation)
- **Treasury revenue:** Vault performance fees → insurance fund singleton
- **Share token standard:** Use standard CAT, not custom NFT (simpler transfers)

---

**Created:** March 6, 2026  
**Foundation Complete:** build-vault-balancer-system.md ✅  
**Estimated Completion:** Q2 2026 (after Rue compiler stabilizes)

---

*Let the vaults auto-compound eternally* ⚖️✨
