# aWizard Familiar 🧙‍♂️

> **Monorepo workspace for Chia DeFi ecosystem** — agent configuration, domain knowledge, quest management, and ALL project source code.

---

## What is aWizard Familiar?

This repository is the **complete development workspace** for the Chia DeFi ecosystem. It contains:

- **🧙 aWizard Agent Configuration** — custom VS Code agent with specialized domain knowledge
- **📚 Skill Library** — 17+ deep reference docs for blockchain, DeFi, Discord, React, and battle systems
- **📋 Quest Management** — Foundation-First workflow system with TODO tracking
- **🧠 Documentation** — Architecture, workflows, agent instructions, theme system
- **⚗️ ALL Projects** — 13 DeFi frontends, battle protocol, Discord Activity, tests, deployment scripts

---

## Repository Strategy: Hybrid Monorepo

**Local:** Keep everything in one folder for development  
**GitHub:** Push selectively to specialized repositories

This workspace syncs to **two GitHub repos**:

1. **[aWizard Familiar](https://github.com/aWizardeth/aWizard-Familiar)** (agent hub)  
   - `.github/`, `docs/`, `.vscode/`, `README.md`
   - Lightweight agent configuration for public sharing

2. **[Forge](https://github.com/aWizardeth/Forge)** (projects)  
   - `projects/`, `tests/`, `scripts/`, shared docs
   - All source code and deployment scripts

**Workflow:** Develop in one monorepo, use `sync-repos.ps1` to push to specialized repos.

See [REPOSITORY_SPLIT_GUIDE.md](REPOSITORY_SPLIT_GUIDE.md) for full hybrid workflow details.

---

## Quick Setup

### Development (Full Monorepo)

```bash
git clone https://github.com/aWizardeth/aWizard-Familiar.git
cd aWizard-Familiar
code .
```

aWizard agent appears in the VS Code agent dropdown automatically. All domain knowledge is in `docs/skills/`, quest workflow in `docs/quests/`, and **all project source code** in `projects/`.

### Install & Run a Project

```bash
cd projects/chia-cfmm
npm install
cp .env.example .env   # add your VITE_WC_PROJECT_ID
npm run dev
```

### Hybrid GitHub Sync (Optional)

To push agent files and projects to separate GitHub repos, see [REPOSITORY_SPLIT_GUIDE.md](REPOSITORY_SPLIT_GUIDE.md).

---

## What's Included

### Agent Definition
- **`.github/agents/awizard.agent.md`** — custom agent with 43 built-in tools
- **`.github/copilot-instructions.md`** — workspace-wide Copilot behavior

### Skills Library

#### Arcane BOW Game
- **`docs/skills/battleKnowledge.md`** — PvE/PvP flows, state channels
- **`docs/skills/apsTierSystem.md`** — scoring, ability unlocks
- **`docs/skills/nftRewards.md`** — soulbound NFT mechanics
- **`docs/skills/aiSeedModel.md`** — deterministic AI, RNG
- **`docs/skills/bondPvpEconomy.md`** — bond escrow, player economy
- **`docs/skills/leaderboardRankings.md`** — on-chain rankings
- **`docs/skills/tournamentSystem.md`** — brackets, matchmaking
- **`docs/skills/discordActivityAuth.md`** — OAuth2, NFT gates
- **`docs/skills/deploymentInfra.md`** — Vercel/VPS architecture
- **`docs/skills/projectArchitecture.md`** — module organization
- **`docs/skills/blockchainDecentralization.md`** — Chia blockchain philosophy
- **`docs/skills/bowAppReference.md`** — WalletConnect/CHIP-0002, state channel, battle protocol, Fighter types
- **`docs/skills/snesWorldEngine.md`** — SNES chunk world, biomes, procedural gen, Godot web export
- **`docs/skills/networkGameplayUX.md`** — spell-cast UX, arcane loaders, WebSocket battles, chain tx progress

#### Chia DeFi
- **`docs/skills/chiaPerpetuals.md`** — on-chain CLOB, perpetuals, liquidations, oracles
- **`docs/skills/nightspireTheme.md`** — Nightspire CSS token system, glow classes, shared palette

### Quest Logs
- **`docs/TODO_DEFI.md`** — Chia DeFi build phases (10 frontends planned, 6 complete)
- **`docs/TODO_WORLD.md`** — SNES world engine quests (FE/BE/SYNC split)
- **`docs/quests/`** — Active quest files (1-2 max)
- **`docs/quests/backlog/`** — Future quests + enhancement backlogs
- **`docs/quests/done/`** — Completed quest foundations
- **`docs/quests/diagrams/`** — Shared Mermaid diagrams

### Quest Workflow System
- **`docs/QUEST_WORKFLOW.md`** — Quick reference cheat sheet
- **`docs/skills/questManagement.md`** — Foundation-First pattern guide

**Pattern:** Build 30-60% foundation in 20 minutes → move to done/ → create enhancement backlog → ship visible progress fast.

### Architecture & Guides
- **`docs/ARCHITECTURE.md`** — Full ecosystem deployment map
- **`docs/NIGHTSPIRE_THEME.md`** — CSS token system for all frontends
- **`docs/AWIZARD_AGENT.md`** — Agent behavior documentation
- **`docs/CODE_AUDIT_REPORT.md`** — Code quality baseline

### Projects (ALL in this repository)
- **`projects/chia-cfmm/`** — Weighted AMM + LP NFT positions (Vite, React 19, Rue)
- **`projects/chia-treasure-chest/`** — On-chain kiosk storefront (Vite, React 19, Rue)
- **`projects/chia-perps/`** — Perpetuals exchange (Vite, React 19, Rue) — Aftermath equiv.
- **`projects/chia-bank/`** — Portfolio aggregation hub (Vite, React 19)
- **`projects/chia-vaults/`** — Auto-rebalancing afLP vaults (Vite, React 19, Rue)
- **`projects/chia-faucet/`** — Testnet faucet (Vite, React 19)
- **`projects/chia-craft/`** — Emoji NFT + token creation (Vite, React 19)
- **`projects/chia-stats/`** — Analytics dashboard (Vite, React 19)
- **`projects/bow-app/`** — Arcane BOW game client (Next.js 16, React 19)
- **`projects/awizard-gui/`** — Discord Activity (Vite, React 19, Discord SDK)
- **`projects/gym-server/`** — PvE battle server (Express, SQLite, TS)
- **`projects/awizard-bot/`** — Discord bot (Discord.js, Node)
- **`projects/arcane-battle-protocol/`** — Protocol spec + contracts (Chialisp, Python, TS)

### Test Suites
- **`tests/cfmm/`** — CFMM math, swap, liquidity, LP NFT, deploy
- **`tests/treasure-chest/`** — Listings, purchase, edge cases, deploy
- **`tests/perps/`** — Accounts, orders, funding, oracle, liquidation, vault
- **`tests/protocol/`** — State channels, bond contracts, gym contracts, NFTs
- **`tests/chips/`** — CHIP-0002, NFT standard compliance

### Deployment Scripts
- **`scripts/deploy.py`** — Deploy projects to Vercel/VPS
- **`scripts/deploy-familiar.py`** — Deploy agent config
- **`sync-repos.ps1`** — Sync files to specialized GitHub repos (optional)

---

## Using the Agent

aWizard operates in specialized mode with deep domain knowledge. Key behaviors:

### Quest Management
- **Foundation First** — build 30-60% viable foundations, ship fast
- **Enhancement Backlogs** — document remaining work for future velocity
- **TODO Phase Tracking** — completion logs in `TODO_DEFI.md`
- See [docs/QUEST_WORKFLOW.md](docs/QUEST_WORKFLOW.md) for the full pattern

### Architecture Awareness
- Knows the full Forge → awizard.dev deployment topology
- Enforces TypeScript strict, React 19 hooks, Tailwind CSS 4
- References skill docs automatically when relevant
- Nightspire theme tokens loaded from `docs/NIGHTSPIRE_THEME.md`

### Model Recommendations
aWizard suggests optimal models for cost efficiency:
- **Claude Sonnet 4** → React components, API integration, docs
- **Claude Opus** → Complex architecture, security analysis, protocol design

---

## Philosophy

> Knowledge and power through transparency — security, decentralization, safety, and magic.

aWizard operates on these principles:
- **Build brick-by-brick** — foundation first, always leave the project buildable
- **Transparency breeds trust** — every decision is documented and verifiable
- **What looks magical is rigorous engineering** — the wizard's secret is discipline

---

## License

MIT

---

## GitHub Repository Strategy

This is a **hybrid monorepo** — all code lives in one local folder for development, but syncs to specialized GitHub repos:

- **[aWizard Familiar](https://github.com/aWizardeth/aWizard-Familiar)** (this repo)  
  Syncs: Agent config (`.github/`), docs (skills/quests), README  
  Purpose: Shareable agent hub, lightweight (<10MB)

- **[Forge](https://github.com/aWizardeth/Forge)**  
  Syncs: All projects, tests, scripts, shared docs  
  Purpose: Complete codebase for collaborators (~500MB)

**Development workflow:** Keep everything in this monorepo, use `sync-repos.ps1` to push selectively.  
**See:** [REPOSITORY_SPLIT_GUIDE.md](REPOSITORY_SPLIT_GUIDE.md) for full hybrid workflow details.

### Legacy Repositories
- **[The Nightspire](https://github.com/aWizardeth/The-Nightspire)** — Original Discord Activity repo (now in `projects/awizard-gui/`)