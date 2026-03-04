# Skill: Project Architecture & Module System

> aWizard's ability to enforce file placement, module boundaries, naming conventions, and the brick-by-brick build methodology.

---

## Domain

aWizard can:

- **Validate file placement** — ensure every file lives in the correct module
- **Enforce naming conventions** — PascalCase for components, camelCase for utilities
- **Guard module boundaries** — prevent cross-contamination between layers
- **Scaffold new files** — generate boilerplate with correct structure
- **Track dependencies** — understand how projects connect

## The Brick-by-Brick Method

> What seems magical to some is simply laying bricks in the right order with the right mortar.

1. **Foundation first** — configs, types, store before UI
2. **One module at a time** — finish one layer before starting the next
3. **Incremental progress** — every commit should leave the project in a working state
4. **Documentation alongside code** — update docs as you build, not after
5. **Skills as building blocks** — each skill is self-contained, composable, referenceable

## Module Map — awizard-gui

| Path              | Purpose                              | Naming           |
| ----------------- | ------------------------------------ | ---------------- |
| `src/components/` | React UI components                 | `PascalCase.tsx` |
| `src/hooks/`      | Custom React hooks                  | `useXxx.ts`      |
| `src/store/`      | Zustand state slices                | `xxxStore.ts`    |
| `src/lib/`        | Utilities, API clients, types       | `camelCase.ts`   |
| `api/`            | Vercel serverless functions         | `camelCase.ts`   |
| `docs/`           | Project management markdown          | `camelCase.md`   |
| `docs/skills/`    | aWizard skill reference files       | `camelCase.md`   |

### Key Files

| File                              | Purpose                                              |
| --------------------------------- | ---------------------------------------------------- |
| `src/hooks/useIsMobile.ts`        | ResizeObserver hook, 480px breakpoint for Discord mobile |
| `src/store/bowActivityStore.ts`   | Top-level app state (auth, user, NFTs)               |
| `src/store/lobbyStore.ts`         | Battle PvP lobby (state channel handshake)           |
| `src/store/chellyzStore.ts`       | Chellyz card game state (AI / Hot Seat / PvP)        |
| `src/store/chellyzLobbyStore.ts`  | Chellyz PvP lobby (deck-based, `bow-chellyz-lobby-v1`) |
| `src/lib/battleEngine.ts`         | Damage calc, turn order, round resolution            |
| `src/lib/fighters.ts`             | `ElementType`, `RarityTier`, `calculateFighterDamage()` |
| `src/lib/stateChannel.ts`         | Chia state channel types + `openChannel()` stub      |
| `src/lib/trackerClient.ts`        | Koba42Corp Redis tracker client                      |
| `api/lobbies.ts`                  | Serverless: create/join/poll lobbies (Redis)         |
| `api/token.ts`                    | Discord OAuth2 token exchange                        |

## Ecosystem Module Map

| Project                  | Entry Point        | Key Modules                          |
| ------------------------ | ------------------ | ------------------------------------ |
| `awizard-gui`            | `src/main.tsx`     | components, hooks, store, lib        |
| `bow-app`                | `app/page.tsx`     | pages, components, providers, store  |
| `gym-server`             | `src/index.ts`     | routes, battle, ai, db, tracker      |
| `arcane-battle-protocol` | n/a                | contracts, gyms, pvp, nft, tests     |

## Convention Quick Reference

- **TypeScript strict** — no `any` without `@ts-expect-error` + reason
- **React 19** — hooks only, no class components
- **Tailwind CSS 4** — no CSS modules, no styled-components
- **Zustand** — no Redux, no Context for global state
- **Vite** — no Webpack
- **Error logging** — prefix all errors with `[aWizard]`

## Source References
- `.github/copilot-instructions.md` — workspace-wide conventions
- `awizard-gui/.github/copilot-instructions.md` — project-level conventions
- `.github/agents/awizard.agent.md` — agent definition
