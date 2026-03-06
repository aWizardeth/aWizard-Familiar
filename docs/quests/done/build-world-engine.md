# Quest: Build World Engine Foundation — `awizard-gui/` Overworld System

> **Assigned to:** World Wizard (this chat)  
> **Forge wizard is:** compiling Rue contracts in `projects/chia-cfmm/`  
> **Perps wizard is:** scaffolding `projects/chia-perps/`  
> **Zero overlap:** All work is in `awizard-gui/src/` — completely different domain

---

## Objective

Build the foundation for the SNES-style overworld engine in the aWizard Discord Activity.  
Scaffold core world types, RNG systems, spell-casting UX components, and store infrastructure.  
**No blockchain or DeFi code** — pure world navigation and gameplay UX.

---

## Context

From `TODO_WORLD.md` Phase 0-1, we need the foundational pieces that enable chunk-based world exploration:
- **World store** — Zustand slice for player position, loaded chunks, encounter state
- **Type system** — `ChunkData`, `BiomeId`, `NpcEntry`, battle integration types
- **Deterministic RNG** — Mulberry32 PRNG for reproducible world generation
- **Spell-cast UX** — `useSpellCast` hook, `ArcaneLoader`, `SpellButton` components
- **Environment config** — World API and Gym server endpoint setup

This unlocks future tile rendering, NPC systems, and encounter→battle bridging.

---

## Steps

### Step 1 — World Types Foundation

Create `src/lib/worldTypes.ts`:
```ts
// Core world data structures
export interface ChunkData {
  x: number;
  y: number;
  biome: BiomeId;
  tiles: number[][]; // 16x16 tile grid (sprite indices)
  collision: boolean[][]; // 16x16 collision mask
  npcs: NpcEntry[];
  encounters: EncounterData[];
  fog?: boolean; // true if chunk not yet explored
}

export type BiomeId = 'wizard_academy' | 'forest' | 'desert' | 'ruins' | 'void';

export interface NpcEntry {
  id: string;
  x: number; // tile position within chunk (0-15)
  y: number;
  spriteKey: string;
  dialogueId: string;
  questId?: string;
}

export interface EncounterData {
  x: number;
  y: number;
  encounterType: 'wild_fighter' | 'gym_challenge' | 'treasure';
  difficulty: number; // 1-5 for AI seed calculation
  reward?: string;
}

export interface BattleResult {
  victory: boolean;
  aps_gained: number;
  nft_reward?: string;
  xp_gained: number;
}
```

---

### Step 2 — Deterministic RNG System

Create `src/lib/rng.ts`:
```ts
// Mulberry32 PRNG - deterministic, fast, good quality
export function mulberry32(seed: number) {
  return function() {
    let t = seed += 0x6D2B79F5;
    t = Math.imul(t ^ t >>> 15, t | 1);
    t ^= t + Math.imul(t ^ t >>> 7, t | 61);
    return ((t ^ t >>> 14) >>> 0) / 4294967296;
  }
}

// World chunk seeding - same chunk always generates same content
export function chunkSeed(chunkX: number, chunkY: number): number {
  return (chunkX * 1000000) + chunkY + 0x12345678;
}

// Random choice from array
export function randomChoice<T>(rng: () => number, array: T[]): T {
  const index = Math.floor(rng() * array.length);
  return array[index];
}

// Random range [min, max)  
export function randomRange(rng: () => number, min: number, max: number): number {
  return Math.floor(rng() * (max - min)) + min;
}
```

---

### Step 3 — World Store (Zustand)

Create `src/store/worldStore.ts`:
```ts
import { create } from 'zustand';
import { ChunkData, BattleResult } from '../lib/worldTypes';

interface WorldState {
  // Player position
  playerChunkX: number;
  playerChunkY: number;  
  playerTileX: number; // position within chunk (0-15)
  playerTileY: number;
  
  // Loaded world data
  loadedChunks: Map<string, ChunkData>;
  currentBiome: string;
  
  // Game state  
  inBattle: boolean;
  battleResult?: BattleResult;
  
  // Actions
  movePlayer: (deltaX: number, deltaY: number) => void;
  loadChunk: (chunkX: number, chunkY: number) => Promise<void>;
  enterBattle: (encounter: string) => void;
  exitBattle: (result: BattleResult) => void;
  
  // Utilities
  getChunkKey: (x: number, y: number) => string;
  getCurrentChunk: () => ChunkData | null;
}

export const useWorldStore = create<WorldState>((set, get) => ({
  // Initial position - Wizard Academy starting area
  playerChunkX: 0,
  playerChunkY: 0,
  playerTileX: 8,
  playerTileY: 8,
  
  loadedChunks: new Map(),
  currentBiome: 'wizard_academy',
  inBattle: false,
  
  movePlayer: (deltaX: number, deltaY: number) => {
    const state = get();
    let newTileX = state.playerTileX + deltaX;
    let newTileY = state.playerTileY + deltaY;
    let newChunkX = state.playerChunkX;
    let newChunkY = state.playerChunkY;
    
    // Handle chunk transitions
    if (newTileX < 0) {
      newChunkX -= 1;
      newTileX = 15;
    } else if (newTileX > 15) {
      newChunkX += 1;
      newTileX = 0;
    }
    
    if (newTileY < 0) {
      newChunkY -= 1;
      newTileY = 15;
    } else if (newTileY > 15) {
      newChunkY += 1;
      newTileY = 0;
    }
    
    set({
      playerTileX: newTileX,
      playerTileY: newTileY,
      playerChunkX: newChunkX, 
      playerChunkY: newChunkY,
    });
  },
  
  loadChunk: async (chunkX: number, chunkY: number) => {
    const key = get().getChunkKey(chunkX, chunkY);
    
    // TODO: Replace with real API call
    const mockChunk: ChunkData = {
      x: chunkX,
      y: chunkY,
      biome: 'wizard_academy',
      tiles: Array(16).fill(null).map(() => Array(16).fill(1)), // grass tiles
      collision: Array(16).fill(null).map(() => Array(16).fill(false)),
      npcs: [],
      encounters: [],
    };
    
    set(state => {
      const newChunks = new Map(state.loadedChunks);
      newChunks.set(key, mockChunk);
      return { loadedChunks: newChunks };
    });
  },
  
  enterBattle: (encounter: string) => {
    set({ inBattle: true, battleResult: undefined });
  },
  
  exitBattle: (result: BattleResult) => {
    set({ inBattle: false, battleResult: result });
  },
  
  getChunkKey: (x: number, y: number) => `${x},${y}`,
  
  getCurrentChunk: () => {
    const state = get();
    const key = state.getChunkKey(state.playerChunkX, state.playerChunkY);
    return state.loadedChunks.get(key) || null;
  },
}));
```

---

### Step 4 — Spell-Cast UX Components

Create `src/components/ArcaneLoader.tsx`:
```tsx
import './ArcaneLoader.css';

interface ArcaneLoaderProps {
  label?: string;
  size?: 'sm' | 'md' | 'lg';
}

export function ArcaneLoader({ label = 'Casting...', size = 'md' }: ArcaneLoaderProps) {
  const sizeClass = {
    sm: 'arcane-loader-sm',
    md: 'arcane-loader-md', 
    lg: 'arcane-loader-lg',
  }[size];
  
  return (
    <div className="arcane-loader">
      <div className={`rune-ring ${sizeClass}`}>
        <div className="rune outer-rune">◊</div>
        <div className="rune inner-rune">◊</div>
      </div>
      <p className="loader-label">{label}</p>
    </div>
  );
}
```

Create `src/components/ArcaneLoader.css`:
```css
.arcane-loader {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

.rune-ring {
  position: relative;
  animation: spin 2s linear infinite;
}

.rune {
  position: absolute;
  color: var(--accent);
  font-weight: bold;
  text-shadow: 0 0 8px var(--accent);
}

.arcane-loader-sm .rune { font-size: 1rem; }
.arcane-loader-md .rune { font-size: 1.5rem; }
.arcane-loader-lg .rune { font-size: 2rem; }

.outer-rune {
  top: -10px;
  left: -10px;
}

.inner-rune {
  top: 5px;
  left: 5px;
  animation: spin 1.5s linear infinite reverse;
}

.loader-label {
  font-size: 0.875rem;
  color: var(--text-secondary);
  margin: 0;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

---

### Step 5 — SpellButton Component

Create `src/components/SpellButton.tsx`:
```tsx
import { ReactNode } from 'react';
import './SpellButton.css';

export type SpellState = 'ready' | 'casting' | 'success' | 'failed';

interface SpellButtonProps {
  children: ReactNode;
  state: SpellState;
  onClick?: () => void;
  disabled?: boolean;
  size?: 'sm' | 'md' | 'lg';
}

export function SpellButton({ 
  children, 
  state, 
  onClick, 
  disabled = false, 
  size = 'md' 
}: SpellButtonProps) {
  
  const getStateClass = () => {
    switch(state) {
      case 'ready': return 'spell-ready';
      case 'casting': return 'spell-casting';
      case 'success': return 'spell-success'; 
      case 'failed': return 'spell-failed';
    }
  };
  
  return (
    <button 
      className={`spell-button ${getStateClass()} spell-button-${size}`}
      onClick={onClick}
      disabled={disabled || state === 'casting'}
    >
      {children}
    </button>
  );
}
```

Create `src/components/SpellButton.css`:
```css
.spell-button {
  position: relative;
  padding: 0.75rem 1.5rem;
  border: 2px solid transparent;
  border-radius: 8px;
  background: var(--bg-deep);
  color: var(--text-primary);
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  overflow: hidden;
}

.spell-button-sm { padding: 0.5rem 1rem; font-size: 0.875rem; }
.spell-button-md { padding: 0.75rem 1.5rem; }
.spell-button-lg { padding: 1rem 2rem; font-size: 1.125rem; }

.spell-ready {
  border-color: var(--accent);
  box-shadow: 0 0 12px rgba(var(--accent-rgb), 0.4);
}

.spell-ready:hover {
  box-shadow: 0 0 20px rgba(var(--accent-rgb), 0.6);
  transform: translateY(-1px);
}

.spell-casting {
  border-color: var(--warning);
  animation: pulse-glow 1s ease-in-out infinite alternate;
  cursor: not-allowed;
}

.spell-success {
  border-color: var(--success);
  box-shadow: 0 0 16px rgba(var(--success-rgb), 0.6);
}

.spell-failed {
  border-color: var(--danger);
  box-shadow: 0 0 16px rgba(var(--danger-rgb), 0.6);
  animation: shake 0.5s ease-in-out;
}

@keyframes pulse-glow {
  from { box-shadow: 0 0 8px rgba(var(--warning-rgb), 0.4); }
  to { box-shadow: 0 0 20px rgba(var(--warning-rgb), 0.8); }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-4px); }
  75% { transform: translateX(4px); }
}
```

---

### Step 6 — useSpellCast Hook

Create `src/hooks/useSpellCast.ts`:
```ts
import { useState } from 'react';

export type SpellState = 'ready' | 'casting' | 'success' | 'failed';

interface SpellCastResult<T> {
  state: SpellState;
  cast: (spellFn: () => Promise<T>) => Promise<T | null>;
  error?: string;
  reset: () => void;
}

export function useSpellCast<T = any>(): SpellCastResult<T> {
  const [state, setState] = useState<SpellState>('ready');
  const [error, setError] = useState<string | undefined>();
  
  const cast = async (spellFn: () => Promise<T>): Promise<T | null> => {
    setState('casting');
    setError(undefined);
    
    try {
      const result = await spellFn();
      setState('success');
      
      // Auto-reset to ready after 2 seconds
      setTimeout(() => setState('ready'), 2000);
      
      return result;
    } catch (err) {
      setState('failed');
      setError(err instanceof Error ? err.message : 'Spell failed');
      
      // Auto-reset to ready after 3 seconds  
      setTimeout(() => setState('ready'), 3000);
      
      return null;
    }
  };
  
  const reset = () => {
    setState('ready');
    setError(undefined);
  };
  
  return { state, cast, error, reset };
}
```

---

### Step 7 — Environment Configuration

Add to `awizard-gui/.env.example`:
```env
# Discord Activity
VITE_DISCORD_CLIENT_ID=your_discord_application_id_here

# World Engine API  
VITE_WORLD_API_URL=http://localhost:3002

# Battle System API
VITE_GYM_SERVER_URL=http://localhost:3001

# Development
VITE_DEV_MODE=true
```

---

### Step 8 — Wire Basic World Navigation

Update `src/App.tsx` to include world navigation test:
```tsx
// Add after existing imports
import { useWorldStore } from './store/worldStore';
import { SpellButton } from './components/SpellButton';
import { useSpellCast } from './hooks/useSpellCast';

// Add inside App component, after activity initialization
const { playerChunkX, playerChunkY, playerTileX, playerTileY, movePlayer } = useWorldStore();
const { state: moveState, cast: castMove } = useSpellCast();

const handleMove = async (dx: number, dy: number) => {
  await castMove(async () => {
    movePlayer(dx, dy);
    // Simulate movement delay
    await new Promise(resolve => setTimeout(resolve, 200));
  });
};

// Add to the JSX (in development section)
<div style={{ marginTop: '2rem', padding: '1rem', border: '1px solid var(--border-color)' }}>
  <h3>World Navigation (Dev)</h3>
  <p>Position: Chunk ({playerChunkX}, {playerChunkY}), Tile ({playerTileX}, {playerTileY})</p>
  
  <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '0.5rem', width: '150px', margin: '1rem 0' }}>
    <div></div>
    <SpellButton state={moveState} onClick={() => handleMove(0, -1)} size="sm">↑</SpellButton>
    <div></div>
    <SpellButton state={moveState} onClick={() => handleMove(-1, 0)} size="sm">←</SpellButton>
    <div></div>
    <SpellButton state={moveState} onClick={() => handleMove(1, 0)} size="sm">→</SpellButton>
    <div></div>
    <SpellButton state={moveState} onClick={() => handleMove(0, 1)} size="sm">↓</SpellButton>
    <div></div>
  </div>
</div>
```

---

## Quest Complete Criteria

- [ ] All type definitions in `src/lib/worldTypes.ts`
- [ ] Deterministic RNG system in `src/lib/rng.ts`  
- [ ] World store with chunk loading in `src/store/worldStore.ts`
- [ ] Spell-cast UX components (`ArcaneLoader`, `SpellButton`)
- [ ] `useSpellCast` hook with proper state management
- [ ] Environment variables configured in `.env.example`
- [ ] Basic world navigation test wired into `App.tsx`
- [ ] All components use Nightspire CSS tokens for theming

**Success metrics:** Player can "move" in the world (updates coordinates), spell-cast UX animates properly, foundation is ready for tile renderer integration.