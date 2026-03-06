# Quest: Build Treasure Chest Frontend — `chia-treasure-chest/` UI

> **Assigned to:** Treasure Wizard (this chat)  
> **Forge wizard is:** compiling Rue contracts in `projects/chia-cfmm/`  
> **Perps wizard is:** scaffolding `projects/chia-perps/`  
> **Zero overlap:** All work is in `chia-treasure-chest/src/` — completely different project

---

## Objective

Connect the existing `chia-treasure-chest` contracts to the UI. Build the React frontend components for viewing chest state, listing items, and purchasing from the on-chain singleton kiosk storefront.

---

## Context

From `TODO_DEFI.md` Phase 2, the treasure chest contracts are already deployed to testnet11, but need UI components:
- **ChestViewer** — displays current chest state (listings, owner, balance)
- **ListItemForm** — owner can list XCH, CAT, or NFT for sale  
- **PurchaseFlow** — buyer workflow: view listing → confirm → submit spend bundle
- **WalletConnect integration** — reuse existing CHIP-0002 pattern from other projects
- **Spacescan links** — display chest singleton coin for transparency

This is pure frontend work — no contract compilation needed!

---

## Steps

### Step 1 — Examine Treasure Chest Architecture

Read the existing contract structure and current UI:
```powershell
# Check current project structure
Get-ChildItem -Recurse .\projects\chia-treasure-chest\ | Select-Object -First 20

# Read contract architecture  
Get-Content .\projects\chia-treasure-chest\contracts\README.md

# Check current App.tsx structure
Get-Content .\projects\chia-treasure-chest\src\App.tsx | Select-Object -First 30
```

---

### Step 2 — Build ChestViewer Component

Create `src/components/ChestViewer.tsx`:
```tsx
interface ChestState {
  owner: string;
  balance: bigint; // Total XCH locked in chest
  listings: ChestListing[];
  singletonCoinId: string;
}

interface ChestListing {
  id: string;
  seller: string;
  asset_type: 'xch' | 'cat' | 'nft';
  asset_id?: string; // CAT id or NFT launcher id
  amount: bigint;
  price: bigint; // in mojos
  listed_at: number; // timestamp
}

export function ChestViewer({ chestAddress }: { chestAddress: string }) {
  // Fetch chest singleton state
  // Display current listings in a table
  // Show chest owner and balance
  // Provide Spacescan link
}
```

---

### Step 3 — Build ListItemForm Component

Create `src/components/ListItemForm.tsx`:
```tsx
interface ListItemFormProps {
  wallet: WalletState;
  chestAddress: string;
  onListSubmit: (listing: NewListing) => Promise<void>;
}

interface NewListing {
  assetType: 'xch' | 'cat' | 'nft';
  assetId?: string;
  amount: bigint;
  price: bigint;
}

export function ListItemForm({ wallet, chestAddress, onListSubmit }: ListItemFormProps) {
  // Form for selecting asset type (XCH/CAT/NFT)
  // Amount input (with wallet balance validation)
  // Price input (in XCH)
  // Submit button that calls onListSubmit
}
```

---

### Step 4 — Build PurchaseFlow Component

Create `src/components/PurchaseFlow.tsx`:  
```tsx
interface PurchaseFlowProps {
  listing: ChestListing;
  wallet: WalletState;
  onPurchase: (listingId: string) => Promise<void>;
}

export function PurchaseFlow({ listing, wallet, onPurchase }: PurchaseFlowProps) {
  // Display listing details
  // Show total cost (price + fees)
  // Confirm/Cancel buttons
  // Loading state during purchase transaction
}
```

---

### Step 5 — Wire Spend Bundle Integration

Create `src/lib/treasureChestSpends.ts`:
```ts
export async function buildListingSpendBundle(
  listing: NewListing,
  walletCoins: CoinRecord[],
  chestSingletonCoin: CoinRecord
): Promise<SpendBundle> {
  // Build chest singleton spend with new listing
  // Include input coins from wallet
  // Return signed spend bundle
}

export async function buildPurchaseSpendBundle(
  listingId: string,
  buyerCoins: CoinRecord[],
  chestState: ChestState
): Promise<SpendBundle> {
  // Build chest singleton spend to execute purchase
  // Transfer asset to buyer, XCH to seller
  // Return signed spend bundle
}
```

---

### Step 6 — Integrate with App.tsx

Update main app with treasure chest functionality:
```tsx
// Add tabs: "Browse Chests", "My Listings", "List Item"
// Wire ChestViewer for browsing mode  
// Wire ListItemForm for sellers
// Wire PurchaseFlow for buyers
// Add wallet connection (reuse existing pattern)
```

---

### Step 7 — Add Chain Integration

Create `src/lib/chestIndexer.ts`:
```ts
export async function fetchChestState(chestAddress: string): Promise<ChestState> {
  // Query chest singleton coin from Chia RPC
  // Parse current state (owner, balance, listings)
  // Return structured data for UI
}

export async function submitSpendBundle(spendBundle: SpendBundle): Promise<string> {
  // Submit to Chia mempool via WalletConnect
  // Return transaction ID for tracking
}
```

---

### Step 8 — Polish & Testing

- Add loading states for all async operations
- Error handling for wallet connection issues  
- Responsive design with Nightspire theme
- Add Spacescan links for all chest transactions
- Test listing and purchasing flows on testnet11

---

## Quest Complete Criteria

- [ ] ChestViewer component displays chest state and listings
- [ ] ListItemForm allows owners to list XCH, CAT, or NFT assets  
- [ ] PurchaseFlow enables buyers to purchase listings
- [ ] Spend bundle builders create proper chest singleton spends
- [ ] WalletConnect integration for all transactions
- [ ] Chain integration fetches real chest state from testnet11
- [ ] Spacescan links for transparency
- [ ] All components use Nightspire CSS theming

**Success metrics:** Users can list items for sale, browse available listings, and purchase items through the on-chain treasure chest kiosk with real Chia transactions.