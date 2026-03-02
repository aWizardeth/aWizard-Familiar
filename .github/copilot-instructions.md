# Workspace Copilot Instructions — Arcane BOW

> Loaded automatically for all Copilot interactions in this workspace.

## Workspace Overview

This workspace contains the **Arcane BOW (Battle of Wizards)** ecosystem — an on-chain PvE/PvP battle game on Chia with soulbound NFTs, state channels, and a Discord-embedded Activity.

## Projects

| Folder                   | Stack                     | Purpose                                  |
| ------------------------ | ------------------------- | ---------------------------------------- |
| `arcane-battle-protocol` | Chialisp, Python, TS      | Protocol spec, contracts, battle engine  |
| `bow-app`                | Next.js 16, React 19      | Game client (wallet, battles, NFTs)      |
| `gym-server`             | Express, SQLite, TS       | PvE gym battle server (port 3001)        |
| `awizard-gui`            | Vite, React 19, Discord SDK | Discord embedded Activity GUI          |

## Coding Conventions

- **TypeScript strict** across all projects. No untyped `any` without explicit `@ts-expect-error`.
- **Tailwind CSS 4** for styling in `bow-app` and `awizard-gui`.
- **Functional React** — hooks only, no class components.
- **Zustand** for client state in React apps.
- File naming: `PascalCase.tsx` for components, `camelCase.ts` for everything else.
- Keep imports explicit — no barrel files / `index.ts` re-exports unless a folder has 5+ modules.

## Architecture Awareness

- The **tracker** (Upstash Redis) is accessed via `bow-app`'s API routes at `/api/tracker/`.
- The **gym-server** runs on port 3001 and serves `/gym/*` endpoints.
- The **aWizard Discord bot** runs on a separate VPS — the `awizard-gui` connects to it via REST.
- Chia wallet interaction goes through **WalletConnect (CHIP-0002)** and the **Sage** wallet.
- Discord Activity authentication uses the **Embedded App SDK** + OAuth2 token exchange.

## Documentation

Each project has its own docs:
- `arcane-battle-protocol/ARCHITECTURE.md` — protocol-level architecture
- `bow-app/STATUS.md` — game client status tracker
- `awizard-gui/docs/` — TODO, IN_DEVELOPMENT, IDEAS, ARCHITECTURE, AWIZARD_AGENT
- `gym-server/docs/protocol/` — protocol spec copies for local reference

When modifying architecture, update the relevant doc alongside the code change.
