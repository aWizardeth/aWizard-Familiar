# Quest Workflow — Quick Reference

> **Foundation First** pattern for maximum velocity

---

## 🎯 One-Page Cheat Sheet

### Folder Structure
```
docs/quests/
├── *.md              # Active (1-2 max)
├── backlog/*.md      # Future + enhancements
├── done/*.md         # Completed foundations
└── diagrams/*.md     # Shared diagrams
```

### Quest Lifecycle
```
backlog/*.md → quests/*.md → done/*.md
                    ↓
            backlog/enhance-*.md
```

---

## ⚡ Quick Commands

### Start a Quest
```powershell
# Move from backlog to active
Move-Item "docs/quests/backlog/build-feature.md" "docs/quests/build-feature.md"
```

### Complete a Quest
```powershell
# 1. Create enhancement backlog (if needed)
# (manually create docs/quests/backlog/enhance-feature.md)

# 2. Move foundation to done
Move-Item "docs/quests/build-feature.md" "docs/quests/done/build-feature.md"

# 3. Update TODO_DEFI.md or TODO_WORLD.md
```

---

## ✅ Foundation Checklist

**Ship when 30-60% complete:**
- [x] Research/design doc
- [x] Math library or core logic
- [x] Project scaffold (`npm run dev` works)
- [x] Core types defined
- [x] README.md comprehensive
- [x] 1-3 key components working

**NOT required:**
- [ ] All UI components
- [ ] Charts/visualizations
- [ ] Smart contracts deployed
- [ ] Testnet deployment
- [ ] Full test coverage

---

## 📝 Quest File Template

### New Quest (in backlog/)
```markdown
# Quest: Build Feature Name

**Status:** BACKLOG
**Created:** YYYY-MM-DD
**Priority:** High/Medium/Low
**Estimated:** X hours

## Objective
One sentence goal.

## Why Now
Context.

## Checklist
- [ ] Step 1
- [ ] Step 2

## Success Criteria
✅ Viable foundation delivered
```

### Completion Summary (add to quest when moving to done/)
```markdown
---

## ✅ Quest Complete — Foundation Delivered

**Status:** FOUNDATION COMPLETE (YYYY-MM-DD)
**Completion:** X%
**Production Ready:** localhost:PORT ✅

**Delivered:**
- ✅ Item 1
- ✅ Item 2

**Remaining:** [enhance-feature.md](../backlog/enhance-feature.md)
```

### Enhancement Quest (in backlog/)
```markdown
# Quest: Enhance Feature Name

**Status:** BACKLOG
**Depends On:** [build-feature.md](../done/build-feature.md) ✅

## Foundation Status ✅
- ✅ Core delivered

## Enhancement Checklist
- [ ] Polish item 1
- [ ] Polish item 2
```

---

## 📊 TODO Update Template

### Progress Summary
```markdown
**✅ Completed Frontends (N phases):**
- Phase X: **Feature** (localhost:PORT) — Foundation ✅
```

### Completion Log
```markdown
- ✅ **YYYY-MM-DD** — Phase X: Feature Foundation ✅ COMPLETE (X%)
  - Quest: [build-feature.md](quests/done/build-feature.md)
  - Enhancements: [enhance-feature.md](quests/backlog/enhance-feature.md)
```

---

## 🚀 Example: 20-Minute Foundation

**Bank of Wizards (60% in 20 min):**

**Minutes 0-5:** Assess what exists
- ✅ Aggregation engine
- ✅ Core components
- ❌ Charts/tabs missing

**Minutes 5-10:** Update quest
- Mark phases complete
- Add completion summary

**Minutes 10-15:** Create enhancement backlog
- Document charts/tabs/oracle work

**Minutes 15-20:** Move files + update TODO
- Move to done/
- Update completion log

**Result:** Quest complete, backlog clear, velocity maximized.

---

## 🎯 Best Practices

### ✅ Do This
- Ship foundations in 20-60 min
- Create enhancement backlog immediately
- Keep active quests ≤ 2
- Update TODO after every completion

### ❌ Don't Do This
- Don't aim for 100% completion
- Don't leave quests in active forever
- Don't skip enhancement backlog
- Don't create mega-quests

---

## 📈 Velocity Metrics

| Complexity | Time | Completion % |
|------------|------|--------------|
| Simple | 10-20 min | 40-50% |
| Medium | 20-40 min | 30-60% |
| Complex | 1-2 hours | 20-40% |

**Recent:** 2 quests / 40 min (March 6, 2026)

---

## 📚 Full Documentation

- **Skill index first:** `docs/skills/README.md`
- **Architecture index first:** `docs/ARCHITECTURE_INDEX.md`
- **Detailed workflow:** `docs/skills/questManagement.md`
- **Backlog README:** `docs/quests/backlog/README.md`
- **Done folder README:** `docs/quests/done/README.md`
- **Agent config:** `.github/agents/awizard.agent.md`

---

**Pattern:** Foundation First → Backlog Later → Maximum Velocity  
**Established:** March 6, 2026  
**aWizard 🧙**
