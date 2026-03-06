# Vault Balancer — Contract Interaction Flow

## 1. Deposit Flow

```mermaid
sequenceDiagram
    participant User
    participant Wallet as Sage Wallet
    participant Frontend as vaults.awizard.dev
    participant Vault as aflp_vault Singleton
    participant LP as CFMM Pool Singleton
    participant Share as vault_share CAT

    User->>Frontend: Click "Deposit 10 XCH"
    Frontend->>Wallet: Request XCH coins via CHIP-0002
    Wallet-->>Frontend: Return spendable coins
    
    Frontend->>Frontend: Build deposit spend bundle
    Note over Frontend: - Vault spend (mode: Deposit)<br/>- User XCH coin spends<br/>- Share CAT mints
    
    Frontend->>Wallet: signCoinSpends(bundle)
    Wallet-->>Frontend: Signed bundle
    
    Frontend->>Wallet: sendTransaction(bundle)
    Wallet->>Vault: Submit spend bundle
    
    Vault->>Vault: Verify deposit amount
    Vault->>LP: Create LP position
    LP-->>Vault: LP NFT minted
    
    Vault->>Share: Mint vault shares
    Note over Vault,Share: shares = (deposit / vault_value) × total_shares
    
    Share-->>User: Vault share CAT tokens
    
    Vault->>Vault: Update state (total_shares += minted)
    
    Frontend->>User: ✅ Deposit confirmed
    Frontend->>User: Display Spacescan link
```

---

## 2. Rebalance Flow (Permissionless)

```mermaid
sequenceDiagram
    participant Watcher as Price Watcher Bot
    participant Oracle as oracle_aggregator Singleton
    participant Frontend as vaults.awizard.dev
    participant Vault as aflp_vault Singleton
    participant LP as CFMM Pool Singleton
    participant Caller as Rebalancer (anyone)

    loop Every 5 minutes
        Watcher->>Oracle: Fetch median price
        Watcher->>LP: Fetch pool spot price
        Watcher->>Watcher: Calculate drift
        
        alt Drift > threshold
            Watcher->>Frontend: Trigger rebalance alert
            Frontend->>Caller: Display opportunity
        end
    end
    
    Caller->>Frontend: Click "Execute Rebalance"
    
    Frontend->>Oracle: Read oracle proof
    Oracle-->>Frontend: { LOVE: 0.010 XCH, sig: ... }
    
    Frontend->>Frontend: Calculate rebalance actions
    Note over Frontend: Target: restore 50/50 weight<br/>Current: 55% LOVE, 45% XCH<br/>Action: Remove LOVE, add XCH
    
    Frontend->>Frontend: Build rebalance spend bundle
    Note over Frontend: - Vault spend (mode: Rebalance)<br/>- Oracle proof in solution<br/>- LP position adjustments<br/>- Caller fee output
    
    Frontend->>Caller: Sign & broadcast
    
    Caller->>Vault: Submit rebalance spend
    
    Vault->>Vault: Verify oracle proof valid
    Vault->>Vault: Verify drift > threshold
    
    Vault->>LP: Swap: remove 5% LOVE, add XCH
    LP-->>Vault: Swap complete, fees collected
    
    Vault->>Vault: Update weights (now 50/50)
    Vault->>Vault: Compound fees → increase share value
    
    Vault->>Caller: Pay rebalancer fee (0.1%)
    
    Frontend->>Frontend: Update vault state
    Frontend->>Caller: ✅ Rebalance executed, earned fee
```

---

## 3. Withdraw Flow

```mermaid
sequenceDiagram
    participant User
    participant Wallet as Sage Wallet
    participant Frontend as vaults.awizard.dev
    participant Vault as aflp_vault Singleton
    participant LP as CFMM Pool Singleton
    participant Share as vault_share CAT

    User->>Frontend: Click "Withdraw 10 afLP shares"
    Frontend->>Wallet: Request vault share CAT coins
    Wallet-->>Frontend: Return share coins
    
    Frontend->>Vault: Fetch current vault state
    Vault-->>Frontend: { reserves, total_shares, ... }
    
    Frontend->>Frontend: Calculate proportional assets
    Note over Frontend: user_ratio = 10 / total_shares<br/>LOVE_out = reserves.LOVE × user_ratio<br/>XCH_out = reserves.XCH × user_ratio
    
    Frontend->>User: Preview: "You'll receive 50 LOVE + 5 XCH"
    User->>Frontend: Confirm withdrawal
    
    Frontend->>Frontend: Build withdrawal spend bundle
    Note over Frontend: - Vault spend (mode: Withdraw)<br/>- Burn share CAT coins<br/>- Create asset outputs to user
    
    Frontend->>Wallet: signCoinSpends(bundle)
    Wallet-->>Frontend: Signed bundle
    
    Frontend->>Wallet: sendTransaction(bundle)
    Wallet->>Vault: Submit spend bundle
    
    Vault->>Vault: Verify shares owned by user
    Vault->>Vault: Calculate withdrawal amounts
    
    Vault->>LP: Reduce LP position (if needed)
    LP-->>Vault: Assets released
    
    Vault->>Share: Burn share CAT tokens
    Vault->>Vault: Update state (total_shares -= burned)
    
    Vault->>User: Transfer proportional assets
    
    Frontend->>User: ✅ Withdrawal confirmed
    Frontend->>User: Display Spacescan link
```

---

## 4. Chest Vault Price Update Flow

```mermaid
sequenceDiagram
    participant Watcher as Floor Watcher Bot
    participant Oracle as floor_oracle API
    participant Frontend as vaults.awizard.dev
    participant ChestVault as chest_vault Singleton
    participant Chest as treasure_chest Singleton

    loop Every 1 hour
        Watcher->>Oracle: Fetch collection floor price
        Oracle-->>Watcher: { floor: 1.2 XCH, timestamp: ... }
        
        Watcher->>ChestVault: Read current listing price
        ChestVault-->>Watcher: { price: 1.1 XCH }
        
        Watcher->>Watcher: Calculate new target price
        Note over Watcher: target = floor × (1 + premium_bps)<br/>= 1.2 × 1.10 = 1.32 XCH
        
        alt Price needs update
            Watcher->>Frontend: Trigger price update
        end
    end
    
    Frontend->>Frontend: Build price update spend bundle
    Note over Frontend: - ChestVault spend (mode: UpdateListing)<br/>- Oracle proof (floor price + sig)<br/>- Chest singleton update
    
    Frontend->>ChestVault: Submit update spend
    
    ChestVault->>ChestVault: Verify oracle proof valid
    ChestVault->>ChestVault: Calculate new listing price
    
    alt abs(floor_change) < stop_loss_threshold
        ChestVault->>Chest: Update listing price
        Chest-->>ChestVault: Price updated (1.32 XCH)
    else floor dropped > stop_loss
        ChestVault->>Chest: Delist (floor crashed)
        Note over ChestVault: Preserve capital, wait for recovery
    else floor spiked > take_profit
        ChestVault->>Chest: Delist (spike detected)
        Note over ChestVault: Opportunity to sell manually higher
    end
    
    ChestVault->>ChestVault: Update state
    Frontend->>Frontend: Display new price
```

---

## 5. Oracle Aggregation Flow

```mermaid
sequenceDiagram
    participant Source1 as Chainlink Oracle
    participant Source2 as Dexie TWAP
    participant Source3 as Tibetswap Pool
    participant Aggregator as oracle_aggregator Singleton
    participant Vault as aflp_vault Singleton

    Source1->>Aggregator: Publish price coin (LOVE: 0.0102 XCH, sig)
    Source2->>Aggregator: Publish price coin (LOVE: 0.0098 XCH, sig)
    Source3->>Aggregator: Publish price coin (LOVE: 0.0100 XCH, sig)
    
    Aggregator->>Aggregator: Verify all BLS signatures
    Aggregator->>Aggregator: Check staleness (< 5 min old)
    Aggregator->>Aggregator: Calculate median(0.0102, 0.0098, 0.0100)
    Note over Aggregator: Median = 0.0100 XCH
    
    Aggregator->>Aggregator: Update state (latest median price)
    
    Vault->>Aggregator: Read oracle price (during rebalance)
    Aggregator-->>Vault: { price: 0.0100, proof: [...sigs...] }
    
    Vault->>Vault: Verify proof
    Note over Vault: - Check >= 3 sources<br/>- Verify BLS sigs<br/>- Check timestamps valid
    
    Vault->>Vault: Use price for rebalance decision
```

---

## 6. Auto-Compounding Flow

```mermaid
sequenceDiagram
    participant LP as CFMM Pool Singleton
    participant Vault as aflp_vault Singleton
    participant Share as vault_share CAT
    participant User

    Note over LP: Users trade on CFMM pool
    LP->>LP: Collect 0.3% swap fees
    
    loop Daily (or on rebalance)
        Vault->>LP: Read accrued fees
        LP-->>Vault: { fees_love: 10, fees_xch: 0.1 }
        
        Vault->>Vault: Reinvest fees into LP position
        Note over Vault: Add fees to reserves → increase TVL
        
        Vault->>Vault: Update share price
        Note over Vault: new_price = (TVL + fees) / total_shares<br/>Users now own more value per share
        
        Vault->>Vault: No new shares minted
        Note over Vault: Share count stays constant,<br/>value per share increases
    end
    
    User->>Vault: Withdraw 10 shares (later)
    Vault->>User: Proportional assets (now worth more)
    Note over User,Vault: Auto-compounded fees are realized
```

---

## Key Contract Invariants

### aflp_vault Singleton

**Assertions on every spend:**
```clojure
(assert (>= (+ deposit_amount current_reserves) MIN_TVL))
(assert (= (+ new_shares total_shares) expected_total))
(assert (<= performance_fee_taken (* pnl performance_fee_bps)))
(assert (verify-oracle-proof oracle_proof oracle_pubkey))
(assert (or (< drift rebalance_threshold) (= spend_mode Rebalance)))
```

**State transitions:**
- `Deposit`: reserves ↑, total_shares ↑, LP positions ↑
- `Withdraw`: reserves ↓, total_shares ↓, LP positions ↓ (if needed)
- `Rebalance`: weights adjust, share_price may ↑ (from arb capture)
- `CompoundFees`: share_price ↑, total_shares unchanged

---

## Security Considerations

### Oracle Manipulation
**Attack:** Attacker publishes fake oracle price to trigger bad rebalance  
**Defense:** Require median of 3+ independent sources, all with valid BLS signatures

### Vault Drain Attack
**Attack:** Malicious rebalancer tries to drain vault via crafted rebalance  
**Defense:** Contract verifies rebalance restores target weights, slippage limits enforced

### Flash Loan Attack
**Attack:** Deposit → trigger rebalance → withdraw to capture arb  
**Defense:** 0.1% withdrawal fee + rebalance cooldown (min 1 hour between rebalances)

### Share Price Manipulation
**Attack:** Large deposit right before fee compound to dilute existing holders  
**Defense:** Fee compounding is continuous (every rebalance), not batched

---

## Gas / Cost Estimates (Chia)

| Operation | CLVM Cost (est.) | Mojos Fee (testnet11) |
|-----------|------------------|----------------------|
| Deposit | ~50M cost | ~0.00005 XCH |
| Withdraw | ~30M cost | ~0.00003 XCH |
| Rebalance | ~100M cost | ~0.0001 XCH |
| Oracle update | ~20M cost | ~0.00002 XCH |

*Note: Actual costs depend on pool complexity and number of assets*

---

**Diagram version:** 1.0  
**Last updated:** March 5, 2026  
**Next:** Implement vaultBalancer.ts math library
