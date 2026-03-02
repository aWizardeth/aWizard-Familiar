# aWizard Familiar 🧙‍♂️

> VS Code agent setup with pre-configured skills, instructions, and development environment for the Arcane BOW ecosystem.

---

## What is aWizard Familiar?

This repository contains everything needed to add **aWizard** as a custom agent to your VS Code workspace. Clone this into any project to get:

- **🧙 aWizard Agent** — appears in your agent dropdown with full tool access
- **📚 11 Skill Files** — deep domain knowledge for blockchain, Discord, React, and battle systems  
- **⚙️ Pre-configured Settings** — auto-approve tools, recommended extensions, consistent dev experience
- **📋 Project Templates** — TODO/IDEAS/IN_DEVELOPMENT docs for project management

---

## Quick Setup

```bash
# Clone into your existing project
git clone https://github.com/aWizardeth/aWizard-Familiar.git
cd aWizard-Familiar

# Copy agent files to your project root
cp -r .github ../
cp -r .vscode ../  
cp -r docs ../

# Open VS Code - aWizard should appear in agent dropdown
code ../
```

**That's it!** aWizard will now be available in the agent picker with full tool access and domain expertise.

---

## What's Included

### Agent Definition
- **`.github/agents/awizard.agent.md`** — custom agent with 43 built-in tools
- **`.github/copilot-instructions.md`** — workspace-wide Copilot behavior

### Skills Library  
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

### Development Environment
- **`.vscode/settings.json`** — auto-approve tools, formatter config
- **`.vscode/extensions.json`** — recommended extensions (Copilot, Prettier, ESLint)

---

## Philosophy

> Knowledge and power through transparency — security, decentralization, safety, and magic.

aWizard operates on these principles:
- **Build brick-by-brick** — foundation first, always leave the project buildable
- **Transparency breeds trust** — every decision is documented and verifiable
- **What looks magical is rigorous engineering** — the wizard's secret is discipline

---

## Model Recommendations

aWizard automatically recommends optimal models for cost efficiency:

- **Claude Sonnet 4** — React components, API integration, docs, standard patterns
- **Claude Opus** — Complex architecture, security analysis, protocol design, debugging

The agent will suggest when to switch models based on task complexity.

---

## License

MIT

---

## Related Projects

- **[The Nightspire](https://github.com/aWizardeth/The-Nightspire)** — Discord Activity GUI
- **[Arcane BOW](https://github.com/your-org/bow-app)** — Web battle client  
- **[Battle Protocol](https://github.com/your-org/arcane-battle-protocol)** — Core game contracts