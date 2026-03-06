# Quest: Bootstrap World Engine Foundation — `awizard-gui/` Scaffolding

> **Assigned to:** World wizard (this chat)  
> **Other wizards are:** Forge (CFMM contracts), Treasure (treasure-chest UI), Perps (project structure)  
> **Zero overlap:** All work is in `awizard-gui/src/` — completely different project from the DeFi work

---

## Objective

Lay the foundation for the SNES-style overworld in `awizard-gui` by scaffolding the core TypeScript infrastructure: Zustand world store, type definitions, RNG system, and environment configuration. This enables future world rendering and chunk loading.

**Time estimate:** 30 minutes — perfect parallel quest!

---

## Context

From `TODO_WORLD.md` Phase 0, there are several **[FE] frontend-only** tasks that need no backend:
- World state management (Zustand store)
- Type definitions for chunks, biomes, NPCs, quests
- Seeded RNG system for procedural generation  
- Environment configuration for world API endpoints

This is pure scaffolding work — no world rendering, no API calls, just TypeScript infrastructure to enable future phases.

**Skills reference:** `docs/skills/snesWorldEngine.md`, `docs/skills/networkGameplayUX.md`

---

## Steps

### Step 1 — World Store Architecture

Create `src/store/worldStore.ts` with Zustand slice for world state:

```typescript
interface WorldState {
  // Player position
  playerPos: { x: number; y: number };
  playerChunk: { chunkX: number; chunkY: number };
  
  // Chunk management
  loadedChunks: Map<string, ChunkData>;
  loadingChunks: Set<string>;
  
  // World state
  currentBiome: BiomeId;
  fogOfWar: boolean;
  
  // Actions
  setPlayerPos: (pos: { x: number; y: number }) => void;
  loadChunk: (chunkX: number, chunkY: number) => Promise<void>;
  unloadChunk: (chunkX: number, chunkY: number) => void;
  updateChunk: (chunkKey: string, data: ChunkData) => void;
}
```

### Step 2 — Type Definitions

Create `src/lib/worldTypes.ts` with core world data structures:

```typescript
// Chunk system
export interface ChunkData {
  x: number;
  y: number;
  biome: BiomeId;
  tiles: TileId[][];           // 16x16 grid
  collisions: boolean[][];     // 16x16 collision layer  
  encounters: EncounterData[]; // Random encounters in this chunk
  npcs: NpcEntry[];           // Fixed NPCs
  exits: ExitPoint[];         // Chunk transitions
}

// Biome definitions  
export type BiomeId = 'forest' | 'desert' | 'ruins' | 'wizard_academy';
export interface BiomeData {
  id: BiomeId;
  name: string;
  tileset: string;
  encounterTable: EncontinueounterData[];
  ambientColor: string;
}

// NPCs and quests
export interface NpcEntry {
  id: string;
  name: string;
  sprite: string;
  pos: { x: number; y: number };
  dialogue: string[];
  questId?: string;
}

export interface QuestData {
  id: string;
  title: string;
  description: string;
  status: 'available' | 'active' | 'completed';
  objectives: { id: string; text: string; completed: boolean; }[];
}
```

### Step 3 — RNG System

Create `src/lib/rng.ts` with seeded pseudorandom number generator:

```typescript
/**
 * Mulberry32 seeded PRNG - fast, good quality, deterministic
 * Used for procedural generation that must be reproducible
 */
export function mulberry32(seed: number): () => number {
  let a = seed;
  return function() {
    a |= 0; a = a + 0x6D2B79F5 | 0;
    let t = Math.imul(a ^ a >>> 15, 1 | a);
    t = t + Math.imul(t ^ t >>> 7, 61 | t) ^ t;
    return ((t ^ t >>> 14) >>> 0) / 4294967296;
  };
}

/**
 * Create RNG instance for world generation
 */
export class WorldRNG {
  private rng: () => number;
  
  constructor(seed: number) {
    this.rng = mulberry32(seed);
  }
  
  next(): number { return this.rng(); }
  nextInt(max: number): number { return Math.floor(this.next() * max); }
  nextRange(min: number, max: number): number { return min + this.next() * (max - min); }
  choose<T>(array: T[]): T { return array[this.nextInt(array.length)]; }
}
```

### Step 4 — Environment Configuration

Update `awizard-gui/.env.example`:

```bash
# Discord Activity (existing)
VITE_DISCORD_CLIENT_ID=your_discord_client_id_here

# World API endpoints (new)
VITE_WORLD_API_URL=http://localhost:3002
VITE_GYM_SERVER_URL=http://localhost:3001

# World generation
VITE_WORLD_SEED=12345
VITE_DEBUG_WORLD=false
```

### Step 5 — Basic World Client

Create `src/lib/worldClient.ts` stub for future API integration:

```typescript  
import { ChunkData, BiomeData } from './worldTypes';

const WORLD_API_URL = import.meta.env.VITE_WORLD_API_URL || 'http://localhost:3002';

export class WorldClient {
  async fetchChunk(chunkX: number, chunkY: number): Promise<ChunkData | null> {
    try {
      const response = await fetch(`${WORLD_API_URL}/world/chunk?x=${chunkX}&y=${chunkY}`);
      if (response.status === 404) return null; // Fog of war
      if (!response.ok) throw new Error(`Chunk fetch failed: ${response.status}`);
      return await response.json();
    } catch (error) {
      console.warn(`Failed to fetch chunk (${chunkX}, ${chunkY}):`, error);
      return null;
    }
  }
  
  async fetchBiome(biomeId: string): Promise<BiomeData | null> {
    try {
      const response = await fetch(`${WORLD_API_URL}/world/biome/${biomeId}`);
      if (!response.ok) return null;
      return await response.json();
    } catch (error) {
      console.warn(`Failed to fetch biome ${biomeId}:`, error);
      return null;
    }
  }
}

export const worldClient = new WorldClient();
```

---

## Success Criteria ✅ COMPLETE

- [x] `worldStore.ts` ✅ **ALREADY COMPLETE** - Advanced Zustand implementation with movement, chunk loading, battles
- [x] `worldTypes.ts` ✅ **ALREADY COMPLETE** - Complete interface system (ChunkData, NpcEntry, EncounterData, etc.) 
- [x] `rng.ts` ✅ **ALREADY COMPLETE** - Mulberry32 PRNG with seeding and utility functions
- [x] `.env.example` ✅ **ALREADY COMPLETE** - Contains `VITE_WORLD_API_URL=http://localhost:3002`
- [x] `worldClient.ts` ✅ **CREATED** - Real API client with chunk/biome fetching + health check
- [x] **INTEGRATED** - worldClient integrated into worldStore with graceful API fallback
- [x] No TypeScript errors in `awizard-gui` ✅ **BUILD PASSES**
- [x] Foundation ready for Phase 1 tile rendering ✅

**Quest Complete:** The world engine foundation was already incredibly advanced! Created the missing API client and integrated it into the existing sophisticated store system ⚡

**Major Discovery:** The `awizard-gui` world system is far more complete than expected - production-ready with chunk streaming, collision detection, battle integration, and sophisticated movement system!