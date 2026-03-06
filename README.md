# aWizard Familiar 🧙‍♂️

> **AI agent configuration and domain knowledge hub** for the Chia DeFi ecosystem.

---

## What is aWizard Familiar?

This repository contains the **agent brain** — specialized VS Code agent configuration with deep domain knowledge:

- **🧙 aWizard Agent** — custom VS Code agent mode with 40+ tools
- **📚 Skill Library** — 17+ domain reference docs (blockchain, DeFi, Discord, React, battle systems)
- **📋 Documentation** — Architecture guides, workflows, theme system, deployment maps
- **🎯 Agent Instructions** — Workspace-wide Copilot behavior and mode definitions

**For project source code**, see **[Forge](https://github.com/aWizardeth/Forge)**.

---

## Quick Setup

```bash
git clone https://github.com/aWizardeth/aWizard-Familiar.git
cd aWizard-Familiar
code .
```

aWizard agent appears in the VS Code agent dropdown automatically. All domain knowledge is loaded from `docs/skills/`.

---

## What's Included

### Agent Definition
- **`.github/agents/awizard.agent.md`** — custom agent with 43 built-in tools
- **`.github/copilot-instructions.md`** — workspace-wide Copilot behavior

### Skills Library

#### BOW Game
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
### Quest Workflow Documentation
- **`docs/QUEST_WORKFLOW.md`** — Quick reference cheat sheet for Foundation-First pattern
- **`docs/skills/questManagement.md`** — Complete workflow guide

### Architecture & Guides
- **`docs/ARCHITECTURE.md`** — Full ecosystem deployment map
- **`docs/NIGHTSPIRE_THEME.md`** — Nightspire CSS token system for all frontends
- **`docs/AWIZARD_AGENT.md`** — Agent behavior and mode documentation
- **`docs/CODE_AUDIT_REPORT.md`** — Code quality baseline and audit results
- **`docs/TODO_DEFI.md`** — DeFi build phases tracking
- **`docs/TODO_WORLD.md`** — World engine development tracking

---

## Using the Agent

aWizard operates as a specialized VS Code agent with deep domain knowledge:

### Agent Modes
- **aWizard mode** — Project management, quest tracking, architecture enforcement
- Auto-loads skills from `docs/skills/` based on context
- Enforces TypeScript strict, React 19 hooks, Tailwind CSS 4
- References Nightspire theme tokens automatically
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