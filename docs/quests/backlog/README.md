# Quest Backlog

This folder contains **two types of quests**:

1. **New feature quests** — Unstarted planning documents for future work
2. **Enhancement quests** (`enhance-*.md`) — Remaining work from completed foundation quests

---

## 🎯 Quest Workflow: Foundation First

**Pattern:** Ship foundations quickly (30-60%), backlog polish for later.

When a quest foundation is complete:
1. Original quest moves to `done/` (foundation delivered)
2. Enhancement quest created here (`enhance-*.md`)
3. TODO updated with completion log

**Example:**
- `done/build-bank-of-wizards.md` — Foundation complete (60%)
- `backlog/enhance-bank-of-wizards.md` — Remaining 40% (charts, tabs, oracle, deployment)

**See:** `docs/skills/questManagement.md` for full workflow documentation.

---

## 📋 New Feature Quests (3 total)

| Quest | Description | Complexity | Dependencies |
|-------|-------------|------------|--------------|
| **bootstrap-aggregator-dex.md** | Smart order router across Tibetswap, Dexie, Splash, Forge | Medium | Multiple DEX integrations |
| **bootstrap-portal-arbitrage.md** | Real-time arbitrage monitor and execution UI | Medium | Aggregator DEX, live price feeds |
| **build-chia-multisig-interface.md** | Gnosis Safe-style multisig wallet for Chia | High | CHIP research, wallet SDK |

---

## 🔧 Enhancement Quests (2 total)

These complete features with viable foundations:

| Quest | Foundation Quest | Completion | Remaining Work |
|-------|------------------|------------|----------------|
| **enhance-bank-of-wizards.md** | [build-bank-of-wizards.md](../done/build-bank-of-wizards.md) ✅ | 60% → 100% | Tabs, charts, quick actions, oracle, deployment |
| **enhance-vault-balancer-system.md** | [build-vault-balancer-system.md](../done/build-vault-balancer-system.md) ✅ | 30% → 100% | Rue contracts, frontend UI, oracle, testnet deployment, CHIP |

---

## ✅ Active Quests

Currently worked on in `docs/quests/`:

| Quest | Status | Progress |
|-------|--------|----------|
| **deploy-testnet-infrastructure.md** | ⚡ IN PROGRESS | 25% (Step 2/8) |

---

## 🔄 Moving Quests

### From Backlog → Active
When ready to start a backlog quest:
```powershell
Move-Item "docs/quests/backlog/quest-name.md" "docs/quests/quest-name.md"
```
1. Update status header to "ACTIVE" or "IN PROGRESS"
2. Add to TODO_DEFI.md or TODO_WORLD.md as appropriate
3. Start building foundation (30-60% target)

### From Active → Done (Foundation Complete)
When quest foundation is viable:
```powershell
# 1. Create enhancement backlog
# Create docs/quests/backlog/enhance-quest-name.md

# 2. Move foundation quest to done
Move-Item "docs/quests/quest-name.md" "docs/quests/done/quest-name.md"
```
1. Add completion summary to quest file
2. Update TODO with ✅ COMPLETE + completion log entry
3. Create `enhance-*.md` in backlog/ for remaining work

---

## 📝 Quest Workflow Best Practices

**✅ Do This:**
- Ship foundations in 20-60 minutes
- Create enhancement backlog immediately
- Keep active quests minimal (1-2 max)
- Update TODO after every completion

**❌ Don't Do This:**
- Don't aim for 100% before moving to done/
- Don't leave quests in active indefinitely
- Don't create mega-quests
- Don't skip the enhancement backlog

**Velocity:** Using this pattern, 2 quests completed in ~40 minutes (March 6, 2026).

---

**Last updated:** March 6, 2026  
**Workflow documented:** `docs/skills/questManagement.md`
- Some may become obsolete as the ecosystem evolves

**Last Updated:** March 6, 2026
