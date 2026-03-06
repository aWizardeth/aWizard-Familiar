# Quest: Implement `perpsmath.ts` — Perpetuals Math Library

> **Assigned to:** Perps wizard (this chat)
> **Forge wizard is:** compiling Rue contracts in `projects/chia-cfmm/`
> **Treasure wizard is:** building `chia-treasure-chest` frontend components
> **Theme wizard is:** validating Nightspire CSS across all 3 DeFi sites
> **Zero overlap:** All work is in `projects/chia-perps/src/lib/perpsmath.ts` only

---

## Objective

Replace the 7 stub functions in `src/lib/perpsmath.ts` with correct, fully-tested
TypeScript implementations that mirror the on-chain perpetuals math from `chiaPerpetuals.md`.
All functions must remain TypeScript strict with `tsc --noEmit` 0 errors.

This is the math engine that every UI component (TradingPanel, PositionPanel, LiquidationPanel)
will call for real-time PnL, liquidation price estimates, and funding payments.

---

## Context

- **Skill reference:** `docs/skills/chiaPerpetuals.md` — authoritative formulas
- **Existing stubs:** `projects/chia-perps/src/lib/perpsmath.ts` — all return zero/placeholder
- **Types:** `projects/chia-perps/src/lib/types.ts` — `PRECISION = 1_000_000_000_000n`
- **No external deps** — pure bigint arithmetic, no float

---

## Math Formulas (from chiaPerpetuals.md)

### Mark Price
$$\text{MarkPrice} = \text{median}(\text{bookMidPrice},\ \text{fundingPrice},\ \text{twap})$$

Sort the 3 bigint inputs, return the middle value.

### Unrealized PnL
- Long: $(\text{markPrice} - \text{entryPrice}) \times \text{size} / \text{PRECISION}$
- Short: $(\text{entryPrice} - \text{markPrice}) \times \text{size} / \text{PRECISION}$

### Margin Ratio
$$\text{marginRatio} = \frac{\text{collateral} + \text{unrealizedPnl}}{\text{notional}}$$
where $\text{notional} = \text{size} \times \text{markPrice} / \text{PRECISION}$

Returns PRECISION-scaled value. Guard against zero notional (return PRECISION if size = 0).

### Liquidation Price

**Long:**
$$L_{\text{long}} = \frac{10000 \times (\text{entryPrice} \times \text{size} - \text{collateral} \times \text{PRECISION})}{(10000 - \text{maintenanceMarginBps}) \times \text{size}}$$

**Short:**
$$L_{\text{short}} = \frac{10000 \times (\text{collateral} \times \text{PRECISION} + \text{entryPrice} \times \text{size})}{(10000 + \text{maintenanceMarginBps}) \times \text{size}}$$

### Funding Rate
$$\text{premiumIndex} = (\text{markPrice} - \text{indexPrice}) \times \text{PRECISION} / \text{indexPrice}$$
$$\text{fundingRate} = \text{clamp}(\text{premiumIndex},\ -\text{clampVal},\ +\text{clampVal})$$
where $\text{clampVal} = \text{clampBps} \times \text{PRECISION} / 10000$

### Funding Payment
$$\text{payment} = \text{size} \times \text{fundingRate} / \text{PRECISION}$$

Longs: positive rate = positive payment (they pay).
Shorts: flip sign.

### Partial Liquidation Size

Find minimum $x$ (units to close) to restore position to initial margin:

$$x_{\min} = \text{size} - \frac{(\text{collateral} + \text{pnl}) \times \text{PRECISION} \times 10000}{\text{initialMarginBps} \times \text{markPrice}}$$

- If $x_{\min} \leq 0$ → position healthy → return `0n`
- If $x_{\min} \geq \text{size}$ → insolvent → return `size` (full liquidation)
- Otherwise return $x_{\min}$

---

## Steps

### Step 1 — Implement `markPrice`
Sort 3 bigint values, return median. Use temp-swap or sort array.

### Step 2 — Implement `unrealizedPnl`
One branch for long/short. Straightforward multiply-divide with PRECISION.

### Step 3 — Implement `marginRatio`
Guard for zero notional. Compute `(collateral + pnl) * PRECISION / notional`.

### Step 4 — Implement `isLiquidatable`
Call `marginRatio`, compare to `BigInt(maintenanceMarginBps) * PRECISION / 10000n`.

### Step 5 — Implement `liquidationPrice`
Use the derived formulas above. Guard for zero size.

### Step 6 — Implement `fundingRate` + `fundingPayment`
PremiumIndex from price spread, clamp, then multiply out.

### Step 7 — Implement `partialLiquidationSize`
Use the $x_{\min}$ formula, clamp to `[0n, size]`.

### Step 8 — Verify
```powershell
cd .\projects\chia-perps
npx tsc --noEmit
```
Expected: 0 errors.

---

## Definition of Done

- [x] `markPrice` — sorts 3 values, returns median ✅
- [x] `unrealizedPnl` — long/short formula, PRECISION-scaled ✅
- [x] `marginRatio` — (collateral + pnl) / notional, zero-guard ✅
- [x] `isLiquidatable` — delegates to marginRatio, bps comparison ✅
- [x] `liquidationPrice` — isolated margin formula for long + short ✅
- [x] `fundingRate` — premiumIndex + symmetric clamp ✅
- [x] `fundingPayment` — size × rate / PRECISION, sign flip for shorts ✅
- [x] `partialLiquidationSize` — minimum close to restore initial margin ✅
- [x] `tsc --noEmit` → 0 errors ✅

## Do NOT touch

- `projects/chia-cfmm/` — forge wizard owns this
- `projects/chia-treasure-chest/` — treasure wizard owns this
- `src/lib/types.ts` — interfaces already complete, no changes needed
- `contracts/*.rue` — contract design is a separate future quest
