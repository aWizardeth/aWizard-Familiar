# Skill: Sage Wallet RPC

> Complete reference for the **Sage Wallet RPC API** — the local wallet RPC that
> powers programmatic access to Sage wallet (v0.12.4+). Covers setup, connection,
> authentication, and every endpoint grouped by domain.

---

## Domain

aWizard can guide use of:

- **Sage RPC setup** — GUI toggle, CLI mode, SSL certificates
- **Authentication flow** — login/logout, key management, mnemonic generation
- **XCH transactions** — send, bulk send, split, combine, multi-send
- **CAT token operations** — issue, send, combine, metadata, resync
- **NFT management** — mint, transfer, assign DID, collections, metadata URIs
- **DID operations** — create, transfer, normalize, update
- **Offer system** — make, take, cancel, combine, import, view
- **Options protocol** — mint, exercise, transfer, update
- **Coin inspection** — list, filter, spendability checks
- **Address management** — derivations, change address, validation
- **Transaction lifecycle** — sign, submit, view, pending transactions
- **Network & sync** — network switching, delta sync, peer management, resync
- **System** — version, database stats, maintenance

---

## Setup

### RPC via GUI

The RPC server runs inside the Sage desktop app process. It is **off by default**.

1. Open Sage → Settings → Advanced
2. Enable "RPC Server"
3. Optionally enable "Start automatically on launch"

The RPC stops when the app closes.

### RPC via CLI

Preferred for headless servers and backend applications.

**Prerequisites:**
- [Tauri prerequisites](https://tauri.app/start/prerequisites/)
- [Rustup](https://rustup.rs/)

**Install:**
```bash
cargo install --git https://github.com/xch-dev/sage --tag v0.11.1 sage-cli
```

**Start the RPC server:**
```bash
sage rpc start
```

**Run commands directly:**
```bash
sage rpc login '{"fingerprint": 2417281}'
```

> **Warning:** Do not run the CLI RPC and the GUI at the same time — they may
> overwrite each other's data.

### Connection Details

| Property         | Value                                |
| ---------------- | ------------------------------------ |
| Default port     | `9257`                               |
| Protocol         | HTTPS (mutual TLS)                   |
| Method           | POST (all endpoints)                 |
| Content-Type     | `application/json`                   |
| Auth             | Client certificate (mTLS)            |

### SSL Certificate

The SSL cert is stored in the Sage data directory under `ssl/`:

| OS      | Path                                                     |
| ------- | -------------------------------------------------------- |
| Windows | `C:\Users\<USER>\AppData\Roaming\com.rigidnetwork.sage\ssl\` |
| macOS   | `~/Library/Application Support/com.rigidnetwork.sage/ssl/` |
| Linux   | `~/.local/share/com.rigidnetwork.sage/ssl/`              |

**Node.js connection example:**
```typescript
import axios from "axios";
import { Agent } from "https";
import fs from "fs";

const sageDir = "..."; // platform-dependent path above

const agent = new Agent({
  rejectUnauthorized: false,
  cert: fs.readFileSync(`${sageDir}/ssl/wallet.crt`),
  key: fs.readFileSync(`${sageDir}/ssl/wallet.key`),
});

axios
  .post("https://localhost:9257/get_sync_status", {}, { httpsAgent: agent })
  .then(console.log)
  .catch(console.error);
```

### Configuration

RPC settings live in `config.toml`:

```toml
[rpc]
enabled = true
port = 9257
```

---

## File Locations

All Sage data is stored in the system data dir:

| File/Dir        | Purpose                                          |
| --------------- | ------------------------------------------------ |
| `config.toml`   | Main app configuration (log level, network, RPC) |
| `keys.bin`      | Binary encoding of all imported keys             |
| `logs`          | Backend log output                               |
| `networks.toml` | Blockchain network definitions                   |
| `peers`         | Binary peer connection history                   |
| `ssl`           | SSL certificates for RPC and full node           |
| `wallets`       | SQLite databases per network and key             |
| `wallets.toml`  | Per-wallet config (delta sync, derivation, etc.) |

---

## API Reference

All endpoints use `POST` method, accept and return JSON.

### Response Codes

| Code | Meaning                                                |
| ---- | ------------------------------------------------------ |
| 200  | Success                                                |
| 400  | Bad request — invalid parameters or malformed JSON     |
| 401  | Unauthorized — no wallet logged in                     |
| 404  | Not found — resource doesn't exist                     |
| 500  | Internal server error                                  |

### Common Types

- **Amount** — string or int64 (in mojos). Use strings for large values.
- **auto_submit** — boolean (default `false`). When true, broadcasts the tx immediately.
- **fee** — Amount. Transaction fee in mojos.

### TransactionResponse

Most write endpoints return:
```json
{
  "coin_spends": [{ "coin": {...}, "puzzle_reveal": "...", "solution": "..." }],
  "summary": { "fee": "...", "inputs": [...] }
}
```

---

## Authentication & Keys

| Endpoint                | Description                                      |
| ----------------------- | ------------------------------------------------ |
| `POST /login`           | Log into wallet by fingerprint (required first)  |
| `POST /logout`          | Log out of current wallet session                |
| `POST /generate_mnemonic` | Generate BIP-39 mnemonic (12 or 24 words)     |
| `POST /import_key`      | Import wallet from mnemonic or private key       |
| `POST /delete_key`      | Permanently delete a wallet key                  |
| `POST /get_key`         | Get key info by fingerprint                      |
| `POST /get_keys`        | List all stored wallet keys                      |
| `POST /get_secret_key`  | Retrieve mnemonic for a fingerprint              |
| `POST /rename_key`      | Change display name of a key                     |
| `POST /set_wallet_emoji`| Set emoji identifier for a wallet                |

**Login — required before most operations:**
```json
POST /login
{ "fingerprint": 1234567890 }
```

**Generate mnemonic:**
```json
POST /generate_mnemonic
{ "use_24_words": false }
// Response: { "mnemonic": "abandon abandon ... about" }
```

**Import key:**
```json
POST /import_key
{
  "name": "My Wallet",
  "key": "abandon abandon ... about",
  "derivation_index": 0
}
```

---

## XCH Transactions

| Endpoint                | Description                                      |
| ----------------------- | ------------------------------------------------ |
| `POST /send_xch`        | Send XCH to an address                          |
| `POST /bulk_send_xch`   | Send XCH to multiple addresses in one tx        |
| `POST /auto_combine_xch`| Auto-combine small XCH coins                    |
| `POST /combine`         | Combine specific coin IDs into one              |
| `POST /split`           | Split a coin into multiple outputs              |
| `POST /multi_send`      | Send XCH + CATs + NFTs in a single tx           |

**Send XCH:**
```json
POST /send_xch
{
  "address": "xch1...",
  "amount": "1000000000000",
  "fee": "100000000",
  "auto_submit": true,
  "memos": ["payment for services"]
}
```

**Multi-send (mixed assets in one tx):**
```json
POST /multi_send
{
  "payments": [
    { "address": "xch1...", "amount": "1000000000000" },
    { "address": "xch1...", "amount": "500000000", "asset_id": "a628c1c2..." }
  ],
  "fee": "100000000",
  "auto_submit": true
}
```

**Split coins:**
```json
POST /split
{
  "coin_ids": ["0xabc..."],
  "output_count": 10,
  "fee": "100000000",
  "auto_submit": true
}
```

---

## CAT Token Operations

| Endpoint                | Description                                      |
| ----------------------- | ------------------------------------------------ |
| `POST /issue_cat`       | Mint a new CAT with specified supply             |
| `POST /send_cat`        | Send CAT tokens to an address                   |
| `POST /bulk_send_cat`   | Send CATs to multiple addresses                  |
| `POST /auto_combine_cat`| Auto-combine CAT coins                           |
| `POST /get_cats`        | List CATs in wallet                              |
| `POST /get_all_cats`    | List all known CATs (including unwatched)         |
| `POST /get_token`       | Get token details by asset ID                    |
| `POST /update_cat`      | Update CAT name, ticker, visibility              |
| `POST /resync_cat`      | Re-fetch CAT metadata from external source       |

**Issue (mint) a CAT:**
```json
POST /issue_cat
{
  "name": "WizardCoin",
  "ticker": "WIZ",
  "amount": "1000000000",
  "fee": "100000000",
  "auto_submit": true
}
```

**Send CAT:**
```json
POST /send_cat
{
  "asset_id": "a628c1c2c6fcb74d...",
  "address": "xch1...",
  "amount": "1000",
  "fee": "100000000",
  "auto_submit": true
}
```

---

## NFT Management

| Endpoint                   | Description                                   |
| -------------------------- | --------------------------------------------- |
| `POST /bulk_mint_nfts`     | Mint multiple NFTs in one tx                  |
| `POST /add_nft_uri`        | Add data/metadata/license URI to NFT          |
| `POST /assign_nfts_to_did` | Assign NFTs to a DID (or unassign with null)  |
| `POST /transfer_nfts`      | Transfer NFTs to a new address                |
| `POST /get_nft`            | Get specific NFT by coin ID                   |
| `POST /get_nfts`           | List NFTs with filtering and pagination       |
| `POST /get_nft_collection` | Get a specific NFT collection                 |
| `POST /get_nft_collections`| List all NFT collections                      |
| `POST /get_nft_data`       | Get raw NFT data file                         |
| `POST /get_nft_icon`       | Get NFT icon image                            |
| `POST /get_nft_thumbnail`  | Get NFT thumbnail                             |
| `POST /update_nft`         | Update NFT display settings                   |
| `POST /update_nft_collection` | Update collection metadata                 |
| `POST /redownload_nft`     | Re-download NFT data from URIs                |

**Bulk mint NFTs:**
```json
POST /bulk_mint_nfts
{
  "did_id": "did:chia:...",
  "fee": "100000000",
  "auto_submit": true,
  "mints": [
    {
      "address": "xch1...",
      "data_uris": ["https://example.com/image.png"],
      "data_hash": "0xabc...",
      "metadata_uris": ["https://example.com/metadata.json"],
      "metadata_hash": "0xdef...",
      "royalty_address": "xch1...",
      "royalty_ten_thousandths": 300,
      "edition_number": 1,
      "edition_total": 100
    }
  ]
}
// Response includes: { "nft_ids": ["nft1..."], "summary": {...}, "coin_spends": [...] }
```

**Add URI to NFT:**
```json
POST /add_nft_uri
{
  "nft_id": "nft1...",
  "uri": "https://mirror.example.com/image.png",
  "kind": "data",
  "fee": "100000000"
}
```
`kind` values: `"data"`, `"metadata"`, `"license"`

**Get NFTs with filtering:**
```json
POST /get_nfts
{
  "offset": 0,
  "limit": 50,
  "include_hidden": false,
  "collection_id": "col1...",
  "sort_mode": "recent"
}
```
Sort modes: `"name"`, `"recent"`

---

## DID Operations

| Endpoint               | Description                                       |
| ---------------------- | ------------------------------------------------- |
| `POST /create_did`     | Create a new DID                                  |
| `POST /get_dids`       | List all DIDs in wallet                           |
| `POST /get_minter_did_ids` | Get DIDs with minting capability              |
| `POST /transfer_dids`  | Transfer DID ownership to new address             |
| `POST /normalize_dids` | Update DIDs to latest on-chain state              |
| `POST /update_did`     | Update DID name and visibility                    |

**Create DID:**
```json
POST /create_did
{
  "name": "My Identity",
  "fee": "100000000",
  "auto_submit": true
}
```

---

## Offer System

| Endpoint                 | Description                                     |
| ------------------------ | ----------------------------------------------- |
| `POST /make_offer`       | Create a peer-to-peer trade offer               |
| `POST /take_offer`       | Accept and complete an offer                    |
| `POST /cancel_offer`     | Cancel an offer on-chain                        |
| `POST /cancel_offers`    | Cancel multiple offers in one tx                |
| `POST /combine_offers`   | Combine multiple offers into one compound offer |
| `POST /get_offer`        | Get offer details by ID                         |
| `POST /get_offers`       | List all offers                                 |
| `POST /get_offers_for_asset` | Get offers involving a specific asset       |
| `POST /import_offer`     | Import an offer from external source            |
| `POST /delete_offer`     | Delete offer from wallet (not on-chain)         |
| `POST /view_offer`       | View offer details without accepting            |

**Make an offer (XCH for CAT):**
```json
POST /make_offer
{
  "offered_assets": [
    { "amount": "1000000000000" }
  ],
  "requested_assets": [
    { "asset_id": "a628c1c2...", "amount": "100" }
  ],
  "fee": "100000000",
  "expires_at_second": 1711152000
}
// Response: { "offer": "offer1...", "offer_id": "0x..." }
```

**Take an offer:**
```json
POST /take_offer
{
  "offer": "offer1...",
  "fee": "100000000",
  "auto_submit": true
}
// Response: { "summary": {...}, "spend_bundle": {...}, "transaction_id": "0x..." }
```

**OfferAmount schema:**
- `amount` — Amount (required)
- `asset_id` — string or null (null = XCH)
- `hidden_puzzle_hash` — optional, for privacy

---

## Options Protocol

| Endpoint                  | Description                                    |
| ------------------------- | ---------------------------------------------- |
| `POST /mint_option`       | Mint a new option with strike + expiration     |
| `POST /exercise_options`  | Exercise in-the-money options                  |
| `POST /get_option`        | Get specific option details                    |
| `POST /get_options`       | List all options with pagination               |
| `POST /transfer_options`  | Transfer options to another address            |
| `POST /update_option`     | Update option visibility                       |

**Mint option:**
```json
POST /mint_option
{
  "underlying": { "amount": "1000000000000", "asset_id": null },
  "strike": { "amount": "500", "asset_id": "a628c1c2..." },
  "expiration_seconds": 86400,
  "fee": "100000000",
  "auto_submit": true
}
```

---

## Coins

| Endpoint                     | Description                                 |
| ---------------------------- | ------------------------------------------- |
| `POST /get_coins`            | List coins with filtering and pagination    |
| `POST /get_coins_by_ids`     | Get specific coins by coin IDs              |
| `POST /get_are_coins_spendable` | Check if coins are currently spendable   |
| `POST /get_spendable_coin_count` | Count of spendable coins               |

**Get coins:**
```json
POST /get_coins
{
  "offset": 0,
  "limit": 100,
  "sort_mode": "amount",
  "ascending": false
}
```
Filter modes: `"all"`, `"selectable"`, `"owned"`, `"spent"`, `"clawback"`
Sort modes: `"coin_id"`, `"amount"`, `"created_height"`, `"spent_height"`, `"clawback_timestamp"`

**CoinRecord shape:**
```json
{
  "coin_id": "0x...",
  "address": "xch1...",
  "amount": "1000000000000",
  "created_height": 5000000,
  "created_timestamp": 1700000000,
  "spent_height": null,
  "spent_timestamp": null,
  "offer_id": null,
  "transaction_id": null,
  "clawback_timestamp": null
}
```

---

## Addresses & Multi-Address Discovery

| Endpoint                       | Description                                         |
| ------------------------------ | --------------------------------------------------- |
| `POST /check_address`          | Validate address and check ownership                |
| `POST /get_derivations`        | List all derived addresses with public keys + hashes |
| `POST /increase_derivation_index` | Extend the derivation window (scan more addresses) |
| `POST /set_change_address`     | Set custom change address                           |

**Check address:**
```json
POST /check_address
{ "address": "xch1..." }
// Response: { "valid": true, "owned": true }
```

**Get derivations — enumerate all wallet addresses:**
```json
POST /get_derivations
{
  "offset": 0,
  "limit": 50,
  "hardened": false,
  "unhardened": true,
  "include_hidden": false
}
// Response:
{
  "derivations": [
    {
      "index": 0,
      "hardened": false,
      "public_key": "0xabc...",    // G1Element hex (96 chars = 48 bytes)
      "puzzle_hash": "0xdef...",   // standard p2 puzzle hash for this key
      "address": "xch1..."         // bech32m encoded address
    }
    // … one entry per derived index in requested range
  ]
}
```

**Increase derivation index — extend address scan window:**
```json
POST /increase_derivation_index
{
  "index": 100,      // new target unhardened index (must be > current)
  "hardened": false
}
// Forces Sage to derive and watch addresses up to the given index.
// Essential before scanning a wallet that may have received coins at high indices.
```

### Multi-Address Scanning Pattern (Operator / Backend)

When building a backend service or operator tool that needs to see all coins across
a wallet's full derivation range:

```typescript
// 1. Login to wallet
await sageRpc("/login", { fingerprint: 1234567890 });

// 2. Check current window size from sync status
const status = await sageRpc("/get_sync_status", {});
const currentIndex = status.unhardened_derivation_index; // e.g. 50

// 3. Optionally extend window if you expect coins at higher indices
if (currentIndex < 100) {
  await sageRpc("/increase_derivation_index", { index: 100, hardened: false });
}

// 4. Enumerate all derived addresses in batches
async function getAllDerivations(batchSize = 50) {
  const all: DerivationRecord[] = [];
  let offset = 0;
  while (true) {
    const { derivations } = await sageRpc("/get_derivations", {
      offset, limit: batchSize, hardened: false, unhardened: true,
    });
    if (!derivations || derivations.length === 0) break;
    all.push(...derivations);
    if (derivations.length < batchSize) break;
    offset += batchSize;
  }
  return all;
}

const derivations = await getAllDerivations();
// derivations[i].address = "xch1..."
// derivations[i].puzzle_hash = "0x..."
// derivations[i].public_key = "0x..."

// 5. Get coins — Sage already aggregates across all watched addresses internally
const { coins } = await sageRpc("/get_coins", { offset: 0, limit: 500, sort_mode: "amount" });
// coins are from ALL derived addresses, no manual enumeration needed for coin listing
```

### Gap Limit Behavior

Chia wallets use a **gap limit** (typically 20 unhardened indices): if the last N
addresses have no coin history, derivation stops. This means:
- A fresh wallet watching indices 0–19 discovers coins at index 0–19 only
- Coins sent to index 50 are invisible until `increase_derivation_index` is called
- After extending, Sage re-syncs and finds the hidden coins on the next sync pass

**Rule of thumb:** Call `increase_derivation_index` during wallet setup if you send
large amounts or expect coins at non-standard indices (e.g., bulk mints, airdrops).

---

## Transactions

| Endpoint                      | Description                                |
| ----------------------------- | ------------------------------------------ |
| `POST /get_transaction`       | Get transaction by height                  |
| `POST /get_transactions`      | List transactions with pagination          |
| `POST /get_pending_transactions` | List pending (unconfirmed) transactions |
| `POST /sign_coin_spends`      | Sign coin spends to create spend bundle    |
| `POST /submit_transaction`    | Submit signed spend bundle to network      |
| `POST /view_coin_spends`      | Preview coin spends without signing        |

**Sign and submit flow:**
```json
// 1. Build a transaction (e.g., send_xch with auto_submit: false)
POST /send_xch
{ "address": "xch1...", "amount": "1000", "fee": "100000000", "auto_submit": false }
// Returns: { "coin_spends": [...], "summary": {...} }

// 2. Sign the coin spends
POST /sign_coin_spends
{ "coin_spends": [...], "auto_submit": false }
// Returns: { "spend_bundle": { "coin_spends": [...], "aggregated_signature": "0x..." } }

// 3. Submit the signed transaction
POST /submit_transaction
{ "spend_bundle": { "coin_spends": [...], "aggregated_signature": "0x..." } }
```

**Partial signing (multi-sig):**
```json
POST /sign_coin_spends
{ "coin_spends": [...], "partial": true }
```

---

## Network & Sync

| Endpoint                     | Description                                  |
| ---------------------------- | -------------------------------------------- |
| `POST /get_network`          | Get current network config                   |
| `POST /get_networks`         | List available networks                      |
| `POST /set_network`          | Switch active network                        |
| `POST /set_network_override` | Per-wallet network override                  |
| `POST /set_delta_sync`       | Enable/disable delta sync                    |
| `POST /set_delta_sync_override` | Per-wallet delta sync override            |

**Get sync status:**
```json
POST /get_sync_status
{}
// Response:
{
  "balance": "1000000000000",
  "unit": { "ticker": "XCH", "precision": 12 },
  "synced_coins": 150,
  "total_coins": 150,
  "receive_address": "xch1...",
  "burn_address": "xch1...",
  "unhardened_derivation_index": 100,
  "hardened_derivation_index": 0,
  "checked_files": 50,
  "total_files": 50,
  "database_size": 1048576
}
```

---

## Peers

| Endpoint                  | Description                                    |
| ------------------------- | ---------------------------------------------- |
| `POST /add_peer`          | Add peer by IP and port                        |
| `POST /remove_peer`       | Remove peer (optionally ban)                   |
| `POST /get_peers`         | List connected peers                           |
| `POST /set_discover_peers`| Enable/disable automatic peer discovery        |
| `POST /set_target_peers`  | Set target peer connection count               |

**Add peer:**
```json
POST /add_peer
{ "ip": "node.example.com:8444" }
```

**Remove and ban:**
```json
POST /remove_peer
{ "ip": "127.0.0.1:8444", "ban": true }
```

---

## System & Database

| Endpoint                         | Description                             |
| -------------------------------- | --------------------------------------- |
| `POST /get_sync_status`          | Sync status, balance, derivation info   |
| `POST /get_version`              | Get Sage version string                 |
| `POST /get_database_stats`       | DB size, page counts, WAL info          |
| `POST /delete_database`          | Delete wallet DB for fingerprint+network|
| `POST /perform_database_maintenance` | Vacuum, analyze, checkpoint WAL     |
| `POST /resync`                   | Full resync with selective data deletion|

**Resync with selective cleanup:**
```json
POST /resync
{
  "fingerprint": 1234567890,
  "delete_coins": true,
  "delete_assets": false,
  "delete_files": false,
  "delete_offers": false,
  "delete_addresses": false,
  "delete_blocks": false
}
```

---

## Themes

| Endpoint                  | Description                                    |
| ------------------------- | ---------------------------------------------- |
| `POST /get_user_theme`    | Get a specific theme NFT                       |
| `POST /get_user_themes`   | List all theme NFTs                            |
| `POST /save_user_theme`   | Save a custom theme NFT                        |
| `POST /delete_user_theme` | Remove a theme NFT                             |

---

## Assets

| Endpoint               | Description                                        |
| ---------------------- | -------------------------------------------------- |
| `POST /is_asset_owned` | Check if a specific asset is owned by this wallet  |

---

## Integration Patterns

### Quick Status Check
```typescript
const status = await sageRpc("/get_sync_status", {});
const balance = BigInt(status.balance); // in mojos
const xch = Number(balance) / 1e12;
```

### Login → Query → Logout Flow
```typescript
await sageRpc("/login", { fingerprint: 1234567890 });
const { cats } = await sageRpc("/get_cats", {});
const { nfts } = await sageRpc("/get_nfts", { offset: 0, limit: 50, include_hidden: false });
await sageRpc("/logout", {});
```

### Build → Sign → Submit Pattern
When `auto_submit: false`, the flow is:
1. Call a write endpoint → get `coin_spends` + `summary`
2. Review the summary (fee, inputs)
3. `POST /sign_coin_spends` → get `spend_bundle`
4. `POST /submit_transaction` → broadcast

This is the pattern for clear-signing UIs and multi-sig workflows.

### Offer Lifecycle
1. `POST /make_offer` → get offer string + ID
2. Share offer string with counterparty
3. Counterparty: `POST /import_offer` → import
4. Counterparty: `POST /take_offer` → complete
5. Or: `POST /cancel_offer` → spend coins to invalidate

---

## Working Rules

- Always call `/login` before any wallet-specific operation
- Amounts are in **mojos** (1 XCH = 1,000,000,000,000 mojos)
- Use `auto_submit: false` for inspection before broadcast
- The RPC uses mutual TLS — you need both `wallet.crt` and `wallet.key`
- `rejectUnauthorized: false` is required because the cert is self-signed
- Do not run GUI and CLI RPC simultaneously
- Default port is 9257 (configurable in `config.toml`)
- NFT royalties use ten-thousandths (300 = 3%)

## Sage vs Chia Node RPC

| Feature              | Sage RPC (this doc)        | Chia Full Node RPC / Coinset |
| -------------------- | -------------------------- | ---------------------------- |
| Port                 | 9257                       | 8555 (full node)             |
| Auth                 | mTLS (wallet cert)         | mTLS or API key              |
| Wallet ops           | Yes (keys, sign, send)     | No                           |
| Block/coin queries   | No (wallet-scoped only)    | Yes (full chain)             |
| Offers               | Yes                        | No                           |
| NFT minting          | Yes                        | No                           |
| Use with frontend    | Via backend proxy           | Direct or via Coinset.org    |

## Source References

- Docs: `https://docs.xch.dev/rpc/setup/`
- OpenAPI spec: `https://github.com/xch-dev/docs/blob/main/src/openapi.json`
- Sage repo: `https://github.com/xch-dev/sage`
- Wallet SDK: `https://github.com/xch-dev/chia-wallet-sdk`
- Sage Discord: `https://discord.gg/sagewallet`
- Config reference: `https://docs.xch.dev/config`
- File locations: `https://docs.xch.dev/files`
