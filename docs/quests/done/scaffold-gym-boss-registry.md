# Quest: Scaffold Gym Boss Registry

**Status:** Active  
**Project:** `bow-app`  
**Priority:** High — foundational to actual game loop  
**Completion Goal:** A real boss registry powering the gym page — typed boss records, element matchups, APS tier gates, and a browsable boss selection UI.

---

## Problem

The `/gym` page currently renders a single generic stub opponent. Every gym battle hits the same hardcoded boss regardless of which gym the player chose, what tier they're attempting, or what APS they've earned. There's no definition of:

- **Which gyms exist** — names, locations, lore  
- **What boss stats are** — HP, ATK, DEF, SPD  
- **What element type the boss uses** — fire, water, grass, etc.  
- **Which tier the player must be** to attempt each gym  
- **What rewards drop** on win (badge mint, wager multiplier)

Without this registry, the gym page is a dead end. The APS leaderboard and NFT badge system have no meaning if the battles themselves are unstructured.

---

## Goals

1. **Create `app/lib/gymRegistry.ts`** — the canonical boss definitions  
2. **Upgrade gym page boss selector** — show available vs locked gyms  
3. **Wire APS tier gates** — lock gyms the player hasn't earned yet  
4. **Show boss element type** — displayed on challenge screen  
5. **Type-safe throughout** — zero `any`, strict interfaces

---

## Implementation Plan

### 1. `app/lib/gymRegistry.ts`

```typescript
export type ElementType =
  | 'fire' | 'water' | 'grass' | 'electric' | 'psychic'
  | 'dark'  | 'ice'   | 'rock'  | 'dragon'   | 'normal';

export interface GymBoss {
  gymId:         string;          // 'pallet', 'cerulean', etc.
  gymName:       string;          // 'Pallet Gym'
  bossName:      string;          // 'Gym Leader Brock'
  element:       ElementType;
  tier:          1 | 2 | 3;       // T1 ≤ APS 24, T2 ≤ APS 49, T3 = 50+
  apsRequired:   number;          // min APS to challenge
  stats: {
    hp:  number;
    atk: number;
    def: number;
    spd: number;
  };
  reward: {
    badgeName:        string;
    wagerMultiplier:  number;     // e.g. 1.5 = 50% bonus on win
  };
  lore: string;                   // one-liner shown on challenge screen
}

export const GYM_REGISTRY: GymBoss[] = [ /* 9 gyms, 3 tiers each */ ];

export function getGymById(gymId: string): GymBoss | undefined
export function getAvailableGyms(aps: number): GymBoss[]
export function getLockedGyms(aps: number): GymBoss[]
```

### 2. Gym Boss Data (9 gyms × up to 3 difficulty tiers)

| Gym         | Element    | T1 APS Gate | T2 Gate | T3 Gate |
|-------------|------------|-------------|---------|---------|
| Pallet      | Normal     | 0           | 10      | 25      |
| Cerulean    | Water      | 5           | 15      | 30      |
| Vermilion   | Electric   | 8           | 18      | 35      |
| Celadon     | Grass      | 10          | 22      | 40      |
| Fuchsia     | Poison     | 12          | 25      | 45      |
| Saffron     | Psychic    | 15          | 28      | 48      |
| Cinnabar    | Fire       | 18          | 32      | 50      |
| Viridian    | Dark       | 20          | 35      | 55      |
| The Nightspire | Dragon  | 30          | 50      | 75      |

Stats scale with tier — Tier 3 Nightspire should feel like an endgame boss.

### 3. Gym Page Upgrade

Replace the single stub with a `GymBossSelector` component:

```tsx
// app/components/GymBossSelector.tsx
// Props: aps: number, onSelect: (boss: GymBoss) => void
// Renders:
//  - Two sections: "Available" (aps >= apsRequired) vs "Locked" (greyed out, shows APS needed)
//  - Each boss card: gym name, element chip, boss name, tier badge, APS gate
//  - Clicking Available card calls onSelect(boss)
//  - Locked cards show unlock requirement ("⚡ Need 15 APS")
```

### 4. Challenge Screen Integration

When a boss is selected, the gym page challenge panel shows:
- Boss name + gym name
- Element type chip (colored by element)
- HP/ATK/DEF/SPD preview (light reveal)
- Reward badge name + wager multiplier
- Lore one-liner in italics  
- `⚔️ Challenge` button → triggers existing battle flow

---

## Deliverables

- [ ] `app/lib/gymRegistry.ts` — typed registry with 9 gyms, helper functions  
- [ ] `app/components/GymBossSelector.tsx` — available/locked boss selection UI  
- [ ] `app/gym/page.tsx` — upgraded to use `GymBossSelector` + challenge panel  
- [ ] `npx tsc --noEmit` → 0 errors  
- [ ] `npm run build` → clean  

---

## Out of Scope

- Actual combat simulation (gym-server handles that)  
- On-chain boss data lookup (mock data is fine for now)  
- NFT fighter stat loading (separate quest: `wire-nft-fighter-gym`)  

---

## Notes

- Element type should use a colour map consistent with Nightspire theme:  
  fire → `#f97316`, water → `#38bdf8`, grass → `#4ade80`, electric → `#facc15`, etc.  
- `apsRequired` should feel fair — Pallet T1 at 0 means anyone can start  
- The Nightspire as the T3 endgame boss is a lore tie-in to the Discord Activity  
