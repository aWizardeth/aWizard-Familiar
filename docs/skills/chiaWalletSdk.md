# Skill: Chia Wallet SDK

> Standard lower-level wallet reference for Chia signer, wallet-engine, and spend-construction work.

Use this skill when a quest drops below WalletConnect session plumbing and needs to reason about how a Chia wallet likely builds, signs, simulates, or serializes spends.

---

## Domain

This skill is for:

- wallet-engine internals
- spend construction and signing behavior
- Rust / WASM / JS / Python binding surfaces
- operator-side or local-service wallet tooling
- understanding the library layer Sage is built on

This skill is not for:

- ordinary frontend WalletConnect integration
- CHIP-0002 session setup and modal behavior
- wallet-specific UX assumptions that only Sage can answer

For frontend WalletConnect flows, start with `bowAppReference.md`.

---

## Core Position

Prefer `xch-dev/chia-wallet-sdk` as the standard lower-level Chia wallet reference when Forge needs a canonical wallet-engine repo.

Reason:

- it is a purpose-built wallet and dApp SDK for Chia
- Sage is built on top of it, so it is the right reference for wallet behavior beneath the Sage RPC / WalletConnect layer
- it provides a cleaner standard source than inventing custom wallet logic from scratch

Repository:

- `https://github.com/xch-dev/chia-wallet-sdk`

---

## What It Is

- a high-performance Chia wallet SDK
- primarily Rust-based
- exposes bindings for additional environments, including WASM and Python
- suitable for wallet apps, dApps, local operators, and signing utilities

## What It Is Not

- not a prebuilt end-user wallet
- not a direct replacement for Sage
- not a substitute for CHIP-0002 method availability in a live wallet session

If the question is “what methods does Sage expose right now?”, inspect Sage or the active session.
If the question is “how should a standard Chia wallet engine construct or sign this?”, use this SDK.

---

## Use It When

Use this skill for quests involving:

- spend bundle construction below the UI layer
- clear-signing or signer-acceptance debugging
- AggSig / aggregated signature behavior
- wallet coin selection and spend-driver logic
- local operator or backend wallet tooling
- evaluating whether Forge should reuse a standard Chia wallet implementation instead of bespoke glue

Examples:

- Forge WalletConnect flow works, but Sage rejects a spend with encoding or signing issues
- a Python or TS helper needs to mirror standard wallet behavior
- a local deployment/operator bridge should use a standard wallet SDK instead of ad hoc spend assembly

---

## Do Not Default To It When

Do not start here for:

- session pairing bugs
- WalletConnect modal issues
- CHIP-0002 namespace or method negotiation
- frontend balance and NFT rendering questions

Start with `bowAppReference.md` for those.

---

## Skill Combinations

Combine this skill with:

- `bowAppReference.md` for WalletConnect and CHIP-0002 surface behavior
- `chiaPrimitivesPatterns.md` for singletons, CATs, and protocol-level asset structure
- `blockchainDecentralization.md` for announcement and trust-model reasoning
- `chiaDevTooling.md` for docs hubs, tracing tools, and operator workflow context

---

## Working Rule For This Repo

When Forge needs a standard Chia wallet implementation reference beyond the browser-session layer:

- prefer `chia-wallet-sdk` over bespoke wallet-engine logic
- treat Sage as the wallet product surface
- treat `chia-wallet-sdk` as the canonical engine/reference layer underneath

That keeps frontend integration, wallet-product behavior, and lower-level signing logic separated cleanly.