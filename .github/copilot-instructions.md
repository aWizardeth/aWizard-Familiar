# Workspace Copilot Instructions — aWizard Familiar

> Loaded automatically for all Copilot interactions in this workspace.

## Workspace Overview

This is the **aWizard Familiar** monorepo — single source of truth for the Arcane BOW + Chia DeFi ecosystem. All projects live under `projects/`. Agent skills live in `docs/skills/`. Test suites in `tests/`. Deploy scripts in `scripts/`.

## Projects

| Folder                                  | Stack                       | Purpose                                          |
| --------------------------------------- | --------------------------- | ------------------------------------------------ |
| `projects/arcane-battle-protocol`       | Chialisp, Python, TS        | Protocol spec, contracts, battle engine          |
| `projects/bow-app`                      | Next.js 16, React 19        | Game client (wallet, battles, NFTs)              |
| `projects/gym-server`                   | Express, SQLite, TS         | PvE gym battle server (port 3001)                |
| `projects/awizard-gui`                  | Vite, React 19, Discord SDK | Discord Activity GUI (The Nightspire)            |
| `projects/awizard-bot`                  | Discord.js, Node            | aWizard Discord bot                              |
| `projects/chia-cfmm`                    | Vite, React 19, Rue/CLVM    | Weighted multi-CAT AMM + LP NFT positions        |
| `projects/chia-treasure-chest`          | Vite, React 19, Rue/CLVM    | On-chain singleton kiosk storefront              |
| `projects/chia-perps` *(planned)*       | Vite, React 19, Rue/CLVM    | On-chain perpetuals exchange (Aftermath equiv.)  |

## Coding Conventions

- **TypeScript strict** across all projects. No untyped `any` without explicit `@ts-expect-error`.
- **Tailwind CSS 4** for styling in `bow-app` and `awizard-gui`.
- **Functional React** — hooks only, no class components.
- **Zustand** for client state in React apps.
- File naming: `PascalCase.tsx` for components, `camelCase.ts` for everything else.
- Keep imports explicit — no barrel files / `index.ts` re-exports unless a folder has 5+ modules.

## Architecture Awareness

- The **tracker** (Upstash Redis) is accessed via `projects/bow-app`'s API routes at `/api/tracker/`.
- The **gym-server** runs on port 3001 and serves `/gym/*` endpoints.
- The **aWizard bot** lives in `projects/awizard-bot/` — `awizard-gui` connects to it via REST.
- Chia wallet interaction goes through **WalletConnect (CHIP-0002)** and the **Sage** wallet.
- Discord Activity authentication uses the **Embedded App SDK** + OAuth2 token exchange.

## Documentation

Each project has its own docs:
- `projects/arcane-battle-protocol/ARCHITECTURE.md` — protocol-level architecture
- `projects/bow-app/STATUS.md` — game client status tracker
- `projects/awizard-gui/docs/` — TODO, IN_DEVELOPMENT, IDEAS, ARCHITECTURE, AWIZARD_AGENT
- `projects/gym-server/docs/protocol/` — protocol spec copies for local reference

When modifying architecture, update the relevant doc alongside the code change.
