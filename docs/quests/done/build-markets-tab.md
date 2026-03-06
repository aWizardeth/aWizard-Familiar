# Quest: Build `MarketsTab.tsx` — Perpetuals Market List

> **Assigned to:** Perps wizard (this chat)
> **Forge wizard is:** compiling Rue contracts in `projects/chia-cfmm/`
> **Treasure wizard is:** building Treasure Chest frontend components
> **Theme wizard is:** validating Nightspire CSS across all 3 DeFi sites
> **Zero overlap:** All work is in `projects/chia-perps/src/` only

---

## Objective

Build `MarketsTab.tsx` — the first real UI component for the Chia Perps exchange.  
Renders a table of perpetual markets with mark price, index price, 24h funding rate,
open interest, and max leverage. Uses mock data now; will swap in on-chain data when
`market_singleton` is deployed to testnet11.

---

## Context

- **Types:** `src/lib/types.ts` — `Market` interface fully defined ✅
- **Math:** `src/lib/perpsmath.ts` — `markPrice`, `fundingRate` implemented ✅
- **App.tsx:** `markets` tab renders a placeholder `glow-card` — replace with real component
- **Nightspire theme:** `src/App.css` — glow-card, glow-text classes available
- No contracts deployed yet → use `MOCK_MARKETS` constant inside the component
- No new deps — Radix UI + Nightspire CSS already present

---

## Steps

### Step 1 — Create `src/components/MarketsTab.tsx`

Table layout with Nightspire styling:
- Column headers: Symbol | Mark Price | Index Price | Funding Rate | Long OI | Short OI | Max Leverage
- Each row: a `Market` from props (or mock data)
- Mark price formatted to 4 decimals in XCH equivalent
- Funding rate formatted as `+0.0123%` / `-0.0042%` with green/red colour
- OI formatted as `1.234 XCH`

### Step 2 — Add `MOCK_MARKETS` seed data

Two mock markets:
- `XCH-USD` — mark 42.50 XCH, index 42.40, funding +0.01%, OI 100/80 XCH, 20x max
- `XCH-BTC` — mark 0.00042 XCH, index 0.00041, funding -0.005%, OI 50/60 XCH, 10x max

### Step 3 — Wire into `App.tsx`

Replace the placeholder `glow-card` in the `markets` tab with `<MarketsTab />`.

### Step 4 — Verify
```powershell
cd .\projects\chia-perps
npx tsc --noEmit
```

---

## Definition of Done

- [x] `src/components/MarketsTab.tsx` exists and renders a market table
- [x] Funding rate cell is green for positive, red for negative
- [x] Mark price and OI formatted correctly (PRECISION-scaled → human-readable)
- [x] `App.tsx` markets tab renders `<MarketsTab />` (no more placeholder text)
- [x] `tsc --noEmit` → 0 errors

## Do NOT touch

- `projects/chia-cfmm/` — forge wizard owns this
- `projects/chia-treasure-chest/` — treasure wizard owns this
- `src/lib/perpsmath.ts` — already complete
- `src/lib/types.ts` — already complete
- `contracts/*.rue` — contract design is a separate future quest
