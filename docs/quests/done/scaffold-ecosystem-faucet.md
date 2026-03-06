# Quest: Scaffold Ecosystem Faucet — `faucet.awizard.dev`

> **Zero overlap guarantee:**
> - `wire-activity-panels.md` → `projects/awizard-gui/` ✅ separate
> - `bootstrap-aggregator-dex.md` → `projects/chia-aggregator/` ✅ separate
> - `build-chia-multisig-interface.md` → multisig project ✅ separate
> - **This quest** → `projects/chia-faucet/` only

---

## Objective

Build the **aWizard Ecosystem Faucet** at `faucet.awizard.dev` — a rate-limited testnet11 token
dispenser so developers and players can bootstrap their wallets for testing:

- **0.25 testnet XCH** per request (enough for fees + pool deposits)
- **500 of each emoji CAT** — LOVE, SPROUT, CASTER, SPELL, POWER, HODL

One request per wallet address per 24 hours. No login required — just a Chia testnet11 address.

---

## Why Now

All six DeFi UIs are live (or nearly live). Every new tester needs a funded wallet to:
- Try swaps on Forge (chia-cfmm)
- Mint emoji tokens on Craft (chia-craft)
- Load up the Bank of Wizards dashboard
- Explore the perps UI (chia-perps)

The faucet is the **on-ramp** for the entire ecosystem.

---

## Stack

| Layer | Choice |
|-------|--------|
| Frontend | Vite 5 + React 19 + TypeScript strict |
| Styling | Tailwind CSS 4 + Nightspire CSS tokens |
| State | Zustand (request status, cooldown timer) |
| Backend | Express + TypeScript, port 3005 |
| Storage | JSON file (cooldowns.json) — no DB needed |
| Port | **5177** (perps=5174, bank=5175, aggregator=5176) |

---

## Steps

### Step 1 — Scaffold `projects/chia-faucet/`

Copy from `chia-craft` template (lightweight, no Radix UI dependency):

```powershell
robocopy projects/chia-craft projects/chia-faucet /E `
  /XD node_modules dist .git `
  /XF "*.lock" /NP /NFL /NDL
```

Then:
- `package.json`: `name = "chia-faucet"`, remove emojiTokens/walletConnect deps
- `vite.config.ts`: port → **5177**
- Clear `src/lib/` — only `faucetClient.ts` needed
- Clear `src/components/` — start fresh

---

### Step 2 — Core Types

**`src/lib/types.ts`**:
```typescript
export interface FaucetToken {
  assetId: string;   // '' for XCH
  symbol: string;
  emoji: string;
  amount: bigint;    // mojos to dispense
}

export interface FaucetRequest {
  address: string;
  requestedAt: number; // Unix timestamp ms
  txIds: string[];     // one per token
  status: 'pending' | 'sent' | 'failed';
}

export interface FaucetStatus {
  eligible: boolean;
  cooldownMs: number;       // ms until next eligible
  lastRequestAt?: number;
  faucetBalances: FaucetToken[];
}
```

---

### Step 3 — Frontend Components

**`src/components/AddressInput.tsx`**
- `<input>` for Chia testnet11 address (`txch1...`)
- Validates: must start with `txch1`, length 62
- Shows inline error for invalid addresses
- `onSubmit(address: string)` callback

**`src/components/FaucetStatus.tsx`**
- Shows per-token amounts being dispensed (emoji + symbol + amount)
- Spinner while request in-flight
- Success: green checkmark + transaction links
- Error: red curse message
- Cooldown countdown: live MM:SS timer

**`src/components/TokenAmountGrid.tsx`**
- 2×3 grid of the 6 emoji tokens + XCH
- Shows how much of each will be dispensed
- Uses Nightspire glow cards

**`src/App.tsx`**
- Layout: centered single-column, max-width 640px
- Header: `🚰 aWizard Faucet` title + `testnet11` badge
- Below: `AddressInput` → `TokenAmountGrid` → `RequestButton`
- After submit: `FaucetStatus` panel replaces button while in-flight

---

### Step 4 — Faucet Client

**`src/lib/faucetClient.ts`**:
```typescript
// POST /api/faucet/request  → FaucetRequest
// GET  /api/faucet/status?address=txch1...  → FaucetStatus

export async function requestTokens(address: string): Promise<FaucetRequest> {
  const res = await fetch('/api/faucet/request', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ address }),
  });
  if (!res.ok) throw new Error(await res.text());
  return res.json();
}

export async function checkStatus(address: string): Promise<FaucetStatus> {
  const res = await fetch(`/api/faucet/status?address=${encodeURIComponent(address)}`);
  if (!res.ok) throw new Error(await res.text());
  return res.json();
}
```

---

### Step 5 — Express Backend (mock for now)

**`server/index.ts`** — returns mock responses so the frontend builds and runs:
```typescript
import express from 'express';
const app = express();
app.use(express.json());

// POST /api/faucet/request
app.post('/api/faucet/request', (req, res) => {
  const { address } = req.body as { address: string };
  if (!address?.startsWith('txch1')) {
    return res.status(400).send('Invalid testnet11 address');
  }
  // TODO: real WalletConnect spend + cooldown check
  res.json({
    address,
    requestedAt: Date.now(),
    txIds: ['mock_tx_1', 'mock_tx_2', 'mock_tx_3'],
    status: 'pending',
  });
});

// GET /api/faucet/status
app.get('/api/faucet/status', (req, res) => {
  res.json({
    eligible: true,
    cooldownMs: 0,
    faucetBalances: [
      { assetId: '', symbol: 'XCH', emoji: '🌿', amount: 250_000_000_000n.toString() },
    ],
  });
});

app.listen(3005, () => console.log('[aWizard Faucet] server :3005'));
```

---

## Acceptance Criteria

- [ ] `npm run build` exits 0 in `projects/chia-faucet/`
- [ ] `tsc --noEmit` exits 0
- [ ] Address input validates `txch1` prefix + 62-char length
- [ ] TokenAmountGrid shows all 7 assets (XCH + 6 CATs)
- [ ] FaucetStatus shows spinner on submit, success panel on mock response
- [ ] Cooldown timer shows MM:SS countdown when `cooldownMs > 0`
- [ ] Nightspire theme CSS tokens loaded (glow-card, glow-text, glow-btn)
