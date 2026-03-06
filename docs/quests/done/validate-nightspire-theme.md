# Quest: Validate & Enhance Nightspire Theme System

> **Assigned to:** Theme wizard (this chat)  
> **Other wizards are:** Forge (CFMM contracts), Perps (scaffolding), Treasure (frontend)  
> **Zero overlap:** Cross-project theme validation + env setup — touches different files

---

## Objective

Validate the Nightspire theme system works consistently across all 3 DeFi frontends (`chia-cfmm`, `chia-treasure-chest`, `chia-perps`), enhance the CSS token system, and ensure proper environment configuration for wallet connections.

**Time estimate:** 30 minutes — perfect parallel quest!

---

## Context

From `TODO_DEFI.md` Phase 0:
- [x] Nightspire CSS tokens added to treasure-chest and cfmm ✅
- [x] Header/footer selectors updated ✅ 
- [x] Perps project scaffolded ✅
- [ ] **Missing:** Verify all 3 sites render with `.glow-card`, `.glow-text`, `.glow-btn` classes

Additional opportunities:
- Standardize `.env.example` files across projects
- Add missing Nightspire utility classes
- Test theme consistency across all frontends

---

## Steps

### Step 1 — Audit Current Theme Implementation

Check what's actually implemented in each project:

```powershell
# Check CFMM theme
Get-Content .\projects\chia-cfmm\src\App.css | Select-String "glow|--accent|--bg-deep"

# Check Treasure Chest theme  
Get-Content .\projects\chia-treasure-chest\src\App.css | Select-String "glow|--accent|--bg-deep"

# Check Perps theme (should inherit from CFMM)
Get-Content .\projects\chia-perps\src\App.css | Select-String "glow|--accent|--bg-deep"
```

---

### Step 2 — Fix Missing Glow Classes

Ensure all 3 projects have complete glow utility classes. Add missing classes:

```css
/* Missing glow utilities (if not present) */
.glow-card {
  box-shadow: 0 0 20px var(--accent-glow);
  border: 1px solid var(--border-color);
  transition: all 0.3s ease;
}

.glow-card:hover {
  box-shadow: 0 0 30px var(--accent-glow), 0 0 60px var(--accent-glow);
  border-color: var(--accent);
}

.glow-text {
  color: var(--accent);
  text-shadow: 0 0 10px var(--accent-glow);
}

.glow-btn {
  background: linear-gradient(135deg, var(--accent), var(--accent-secondary));
  color: var(--text-primary);
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  box-shadow: 0 0 20px var(--accent-glow);
  transition: all 0.3s ease;
}

.glow-btn:hover {
  box-shadow: 0 0 30px var(--accent-glow);
  transform: translateY(-2px);
}
```

---

### Step 3 — Standardize Environment Files

Ensure all projects have consistent `.env.example` with proper WalletConnect setup:

**Required in all 3 `.env.example` files:**
```bash
# Wallet Connect v2 (get from https://cloud.walletconnect.com)
VITE_WC_PROJECT_ID=your_project_id_here

# Chia Network (testnet11 for development)  
VITE_CHIA_NETWORK=testnet11

# Development URLs
VITE_GYM_SERVER_URL=http://localhost:3001
VITE_WORLD_API_URL=http://localhost:3002
```

---

### Step 4 — Test Theme Rendering

Start each project and verify glow classes work:

```powershell
# Test CFMM (port 5173)
cd projects/chia-cfmm
npm run dev

# Test Treasure Chest (port 5175) 
cd projects/chia-treasure-chest  
npm run dev

# Test Perps (port 5174)
cd projects/chia-perps
npm run dev
```

Use browser dev tools to:
1. Add `.glow-card` class to main containers
2. Add `.glow-text` class to headers  
3. Add `.glow-btn` class to buttons
4. Verify colors use CSS variables (`--accent`, `--bg-deep`, etc.)

---

### Step 5 — Document Theme Usage

Create `docs/NIGHTSPIRE_THEME.md` with:
- CSS variable reference
- Glow class examples  
- Cross-project consistency guidelines
- Color palette specification

---

## Success Criteria ✅ COMPLETE

- [x] All 3 projects have complete glow utility classes ✅
- [x] All 3 projects have standardized `.env.example` ✅  
- [x] Theme renders consistently across all frontends ✅
- [x] Documentation created at `docs/NIGHTSPIRE_THEME.md` ✅
- [x] No overlap with other ongoing quests ✅

**Quest Complete:** Phase 1 wallet testing across all projects is now ready ⚡