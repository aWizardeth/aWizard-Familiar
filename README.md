# aWizard Familiar 🧙‍♂️

> **AI agent configuration and domain knowledge hub** for the Chia DeFi ecosystem.

---

## What is aWizard Familiar?

This repository contains the **agent brain** — specialized VS Code agent configuration with deep domain knowledge:

- **🧙 aWizard Agent** — custom VS Code agent mode with 40+ tools
- **📚 Skill Library** — 17+ domain reference docs (blockchain, DeFi, Discord, React, battle systems)
- **📋 Documentation** — Architecture guides, workflows, theme system, deployment maps
- **🎯 Agent Instructions** — Workspace-wide Copilot behavior and mode definitions

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

### Architecture & Guides
- **`docs/ARCHITECTURE.md`** — Full ecosystem deployment map
- **`docs/NIGHTSPIRE_THEME.md`** — Nightspire CSS token system for all frontends
- **`docs/AWIZARD_AGENT.md`** — Agent behavior and mode documentation
- **`docs/CODE_AUDIT_REPORT.md`** — Code quality baseline and audit results
- **`docs/TODO_DEFI.md`** — DeFi build phases tracking
- **`docs/TODO_WORLD.md`** — World engine development tracking
- **`docs/QUEST_WORKFLOW.md`** — Foundation-First pattern quick reference
- **`docs/skills/questManagement.md`** — Complete quest workflow guide

---

## Using the Agent

aWizard operates as a specialized VS Code agent with deep domain knowledge:

- **aWizard mode** — Project management, architecture enforcement, domain expertise
- Auto-loads skills from `docs/skills/` based on context
- Enforces TypeScript strict, React 19 hooks, Tailwind CSS 4, Zustand for state
- References Nightspire theme tokens and architecture patterns automatically
- Suggests optimal models: **Sonnet 4** for standard work, **Opus** for complex architecture

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