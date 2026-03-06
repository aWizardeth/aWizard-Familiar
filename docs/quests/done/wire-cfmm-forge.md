# Quest: Wire the CFMM Forge — `projects/chia-cfmm/`

> **Assigned to:** Forge wizard (this chat)
> **Perps wizard is:** scaffolding `projects/chia-perps/` — no shared files
> **Zero overlap:** All work is in `projects/chia-cfmm/` only

---

## Objective

Get `forge.awizard.dev` functional on testnet11 — real swap spend bundles submitted through the Sage wallet, starting with the Swap UI flow.

---

## Context

- Contracts: `projects/chia-cfmm/contracts/`
- Frontend: `projects/chia-cfmm/src/`
- Rue compiler: `rue build -x <file>` from `projects/chia-cfmm/`
- Wallet: CHIP-0002 / Sage on testnet11 — `walletConnect.ts` ✅
- Bundle builder: `src/lib/spendBundles.ts` — `buildSwapBundle` complete ✅ but `PUZZLE_REVEALS` are `null`

---

## Steps

### Step 1 — Fix `pool_singleton.rue` compilation ✅ IN PROGRESS

**Issues found:**
1. Five `AssetSlot { ...state.xxx, reserve: newVal }` struct spreads — fixed ✅
2. `PoolState { ...state, key: val }` spreads — fixed ✅
3. `fn(...sol: Any) -> List<Condition>` variadic fn type in `SetFee` — fixed ✅
4. `m.admin_solution(...m.solution)` variadic call — fixed ✅

**Verify with:**
```powershell
cd projects/chia-cfmm
rue build -x contracts/pool_singleton.rue
```

---

### Step 2 — Fix `lp_nft.rue` compilation

```powershell
rue build -x contracts/lp_nft.rue
```

Likely the same struct-spread issues. Fix the same way — explicit field copies.

---

### Step 3 — Fill `PUZZLE_REVEALS` in `spendBundles.ts`

After successful compilation:
```powershell
rue build -x contracts/pool_singleton.rue  # copy hex output
rue build -x contracts/lp_nft.rue          # copy hex output
```

Fill into `src/lib/spendBundles.ts`:
```ts
export const PUZZLE_REVEALS = {
  POOL_SINGLETON_BASE: '<hex from pool_singleton>',
  LP_NFT_BASE: '<hex from lp_nft>',
};
```

---

### Step 4 — Wire `SwapPanel` onClick

**Current state:**
- `SwapPanel.tsx` — Swap button has no `onClick`, props: `{ pool, connected }`
- `App.tsx` — calls `<SwapPanel pool={pool} connected={wallet.connected} />`

**Changes needed:**

`SwapPanel.tsx` — add `onSwap` prop:
```tsx
interface SwapPanelProps {
  pool: PoolState;
  connected: boolean;
  onSwap: (inIndex: number, outIndex: number, amountIn: bigint, minAmountOut: bigint) => Promise<void>;
}
```

Add loading/error state and wire the button:
```tsx
const [swapping, setSwapping] = useState(false);
const [swapError, setSwapError] = useState<string | null>(null);

const handleSwap = async () => {
  setSwapping(true);
  setSwapError(null);
  try {
    await onSwap(inIndex, outIndex, amountIn, minOut);
  } catch (e) {
    setSwapError(e instanceof Error ? e.message : 'Swap failed');
  } finally {
    setSwapping(false);
  }
};

<Button onClick={handleSwap} disabled={!connected || !quote || amountIn === 0n || swapping}>
  {swapping ? 'Swapping…' : !connected ? 'Connect Wallet to Swap' : 'Swap'}
</Button>
```

`App.tsx` — pass `onSwap` callback:
```tsx
const { wallet, connect, disconnect, actions } = useChiaWallet();

const handleSwap = async (inIndex: number, outIndex: number, amountIn: bigint, minAmountOut: bigint) => {
  if (!wallet.address) throw new Error('Wallet not connected');
  await buildSwapBundle({
    pool,
    senderPuzzleHash: wallet.address,
    inIndex,
    outIndex,
    amountIn,
    minAmountOut,
    recipientPuzzleHash: wallet.address,
    walletActions: actions,
  });
};

<SwapPanel pool={pool} connected={wallet.connected} onSwap={handleSwap} />
```

---

### Step 5 — Wire `LiquidityPanel` submit

Same pattern — add `onDeposit` and `onWithdraw` callbacks.
Uses `buildMultiAssetDepositBundle` and `buildWithdrawBundle` from `spendBundles.ts`.

---

## Definition of Done

- [ ] `rue build -x contracts/pool_singleton.rue` exits 0
- [ ] `rue build -x contracts/lp_nft.rue` exits 0
- [ ] `PUZZLE_REVEALS` filled with real hex in `spendBundles.ts`
- [ ] SwapPanel Swap button calls `buildSwapBundle` → `actions.signAndSend`
- [ ] SwapPanel shows loading state during swap and surfaces errors
- [ ] LiquidityPanel Deposit/Withdraw buttons wired
- [ ] No TypeScript errors (`tsc --noEmit`)

## Do NOT touch

- `projects/chia-perps/` — perps wizard owns this
- `projects/chia-treasure-chest/` — separate quest
- `docs/skills/` — read-only reference
- `tests/` — test wizard owns this
