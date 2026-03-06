# Skill: SNES-Style Expanding World Engine

> aWizard's reference for building a Pokémon/Zelda-style tile-based overworld in the browser
> — chunk streaming, biome design, procedural generation, NPC/encounter logic,
> and integration with the Arcane BOW battle system.

---

## Domain

aWizard can:

- **Scaffold the world data layer** — chunk JSON schema, biome registry, NPC manifest
- **Design a procedural generation pipeline** — seeded RNG, biome weighting, encounter zones
- **Integrate overworld encounters with the battle system** — trigger gym / PvP flows from world events
- **Build tile-based movement in the browser** — canvas or Phaser 3 renderer inside the Discord Activity
- **Reference proven SNES-era engine patterns** from community open-source repos

---

## Reference Repositories

| # | Repo | Stack | Why It Matters |
|---|------|-------|----------------|
| 1 | [godot-pokemon](https://github.com/samsface/godot-pokemon) | Godot 4, GDScript | Overworld structure, battle system, trainer/monster/move tables |
| 2 | [OpMon](https://github.com/OpMonTeam/OpMon) | C++, SFML | Modular monster-catching framework; world scenes + object system |
| 3 | [pokemon-essentials](https://github.com/Maruno17/pokemon-essentials) | RPG Maker XP | Full event system, encounter tables, NPC logic, map metadata |
| 4 | [PokemonGodotEssentials](https://github.com/pokemon-essentials/PokemonGodotEssentials) | Godot 4 | Godot port — tile world + battle open architecture |
| 5 | [godot-2d-tactical-rpg-movement](https://github.com/gdquest-demos/godot-2d-tactical-rpg-movement) | Godot 4 | Grid movement, A* pathfinding, unit AI — great for tactical dungeon combat |
| 6 | [Pixelorama](https://github.com/Orama-Interactive/Pixelorama) | Godot 4 | Pixel sprite pipeline — sprite sheets, tilesets, animation export |
| 7 | [godot-demo-projects](https://github.com/godotengine/godot-demo-projects) | Godot 4 | `2d/procedural_generation` + `2d/tilemap` — chunk streaming demos |
| 8 | [overworld-godot](https://github.com/DavidCobb/overworld-godot) | Godot 4 | Zelda SNES overworld, NPC dialogue, tilemap transitions |
| 9 | [minetest](https://github.com/minetest/minetest) | C++, Lua | Infinite chunk loading algorithm, mod API, world persistence |
| 10 | [awesome-godot](https://github.com/godotengine/awesome-godot) | Index | Aggregated tilemap tools, RPG frameworks, plugin ecosystem |

---

## Target Renderer (Browser / Discord Activity)

Godot's **web export** is the canonical choice for the Discord Activity because:
- Produces a single `index.html` + WASM bundle importable into the Activity iframe
- Supports full tilemap + physics + SNES-resolution (256×224) scaled output
- GDScript is AI-friendly and fast to iterate
- The embedded app SDK can be proxied via `postMessage` from the host page to Godot's JS bridge

Alternative (pure web): **Phaser 3** + `@discord/embedded-app-sdk` when you want zero Godot overhead.

---

## World Data Architecture

```
world/
  registry.json          ← biome list, spawn weights, encounter tables
  chunks/
    chunk_0_0.json
    chunk_0_1.json
    chunk_1_0.json
  npcs/
    npc_manifest.json    ← NPC id, position, dialogue tree ref, quest id
  quests/
    quest_manifest.json
  dungeons/
    dungeon_manifest.json
  biomes/
    forest.json
    desert.json
    ruins.json
    wizard_academy.json
```

### Chunk Schema (`chunk_x_y.json`)

```json
{
  "id": "chunk_0_0",
  "x": 0,
  "y": 0,
  "biome": "forest",
  "tiles": [[0,1,1,0], [1,2,2,1]],
  "encounters": [
    { "zone": [2,2,6,6], "type": "gym_pve", "gym_id": "forest_gym_01", "rate": 0.15 }
  ],
  "npcs": ["npc_wandering_mage_01"],
  "exits": {
    "north": "chunk_0_1",
    "east":  "chunk_1_0",
    "south": null,
    "west":  null
  }
}
```

### Biome Schema (`forest.json`)

```json
{
  "id": "forest",
  "tileset": "tileset_forest.png",
  "bg_music": "forest_theme.ogg",
  "encounter_pool": ["sprite_moth", "sprite_deer", "sprite_goblin"],
  "gym_difficulty_range": [1, 3],
  "loot_table": ["xp_small", "mana_crystal", "scroll_common"],
  "fog_density": 0.2,
  "visual_filter": "none"
}
```

---

## Procedural Generation Pipeline

```
seed (deterministic, from player wallet hash or server seed)
  ↓
biome_noise(x, y)          ← simplex noise → biome type per chunk
  ↓
tile_generator(biome)      ← weighted tile selection from biome tileset
  ↓
encounter_placer()         ← scatter encounter zones per biome rules
  ↓
npc_placer()               ← weighted NPC spawn per biome
  ↓
exit_connector()           ← link adjacent chunks
  ↓
serialize → chunk_x_y.json
```

### Seeded RNG (TypeScript)

```ts
// Mulberry32 — fast, deterministic, seedable
function mulberry32(seed: number) {
  return function () {
    seed |= 0; seed = (seed + 0x6d2b79f5) | 0;
    let t = Math.imul(seed ^ (seed >>> 15), 1 | seed);
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t;
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
}
```

Always derive the world seed from the **AI seed model** (`aiSeedModel.md`) so the world and battles share the same deterministic root.

---

## Biome Definitions

| Biome | SNES Aesthetic | Encounter Types | Gym Style |
|-------|---------------|-----------------|-----------|
| `forest` | Zelda LttP Kakariko woods | Wild encounters, merchant NPCs | Druid gym — nature magic |
| `desert` | Dragon Quest sandy wastes | Ambush encounters, buried treasures | Sandstorm gym — earth/fire |
| `ruins` | Final Fantasy ancient temple | High-difficulty encounters, puzzle gates | Curse gym — dark magic |
| `wizard_academy` | Hogwarts meets Ys | Tutorial, PvP arena zone, leaderboard board | Arcane gym — all elements |
| `nightspire` | Gothic castle exterior | Boss encounters only, APS-gated | Champion gym — Tier 5+ only |

---

## Player Movement & Chunk Loading

### Grid-Based Movement (Phaser 3 / Godot)

- Tile size: **16×16 px** (scaled ×3 = 48×48 display px for modern screens)
- Player moves on a discrete grid — no sub-pixel physics
- Movement input is queued, not instant (SNES feel: slight slide into next tile)

### Dynamic Chunk Streaming

```
Player position → current_chunk(x, y)
  ↓
Preload radius: chunks within Manhattan distance 2 (3×3 grid around player)
  ↓
Load chunk JSON via API route or Godot ResourceLoader
  ↓
Unload chunks outside radius 4 (GC / cache evict)
```

**API route (awizard-gui / bow-app):**
```
GET /api/world/chunk?x=0&y=1
```

---

## NPC System

### NPC Manifest Entry

```json
{
  "id": "npc_wandering_mage_01",
  "name": "Aldric the Wanderer",
  "sprite": "npc_mage_blue.png",
  "chunk": "chunk_0_0",
  "pos": { "x": 5, "y": 3 },
  "dialogue_tree": "aldric_intro",
  "quest_id": "quest_mana_crystal_01",
  "shop": null
}
```

### Dialogue Tree Format (simplified ink-style)

```
=== aldric_intro ===
Aldric: Traveler! You've entered the Whispering Forest.
  * [What is this place?]
    Aldric: A battleground. Wizards fight here for APS glory.
    → aldric_quest_offer
  * [I need to train.]
    Aldric: Find the Druid Gym to the east.
    → END
```

---

## Battle Encounter Integration

When the player walks into an encounter zone:

```
1. Roll encounter_rate → if triggered:
2. Freeze overworld (fade out)
3. Read zone.type:
   - "gym_pve"  → open gym battle flow (battleKnowledge.md PvE flow)
   - "wild"     → open wild encounter (lightweight, no state channel)
   - "boss"     → open boss flow (APS gate check first)
   - "pvp_arena"→ enter PvP matchmaking lobby (bondPvpEconomy.md)
4. Battle resolves → return result to overworld
5. Fade in → update player state (APS, HP, inventory)
```

---

## Quest System

```json
{
  "id": "quest_mana_crystal_01",
  "title": "Mana Crystal Collection",
  "giver_npc": "npc_wandering_mage_01",
  "steps": [
    { "type": "collect", "item": "mana_crystal", "count": 3 },
    { "type": "return_to_npc", "npc_id": "npc_wandering_mage_01" }
  ],
  "reward": { "xp": 500, "item": "scroll_uncommon", "aps_bonus": 10 }
}
```

Quest progress is tracked in the player's session store (Zustand `worldStore.ts`).

---

## Zustand World Store Outline

```ts
// src/store/worldStore.ts
interface WorldState {
  currentChunk: { x: number; y: number };
  loadedChunks: Record<string, ChunkData>;
  playerPos: { x: number; y: number };
  activeQuests: Quest[];
  completedQuests: string[];
  inventory: InventoryItem[];
  encounterPending: EncounterData | null;

  actions: {
    movePlayer: (dx: number, dy: number) => void;
    loadChunk: (x: number, y: number) => Promise<void>;
    triggerEncounter: (data: EncounterData) => void;
    resolveEncounter: (result: BattleResult) => void;
    acceptQuest: (quest: Quest) => void;
    completeQuest: (questId: string) => void;
  };
}
```

---

## Godot Web Export → Discord Activity

```
Godot project/
  ├ export_presets.cfg   ← Web HTML5 export, VRAM compression off
  ├ web_export/
  │   ├ index.html       ← loaded by Discord Activity iframe
  │   ├ game.js
  │   └ game.wasm
```

Bridge Godot ↔ Discord SDK via `JavaScriptBridge`:
```gdscript
# In Godot (GDScript)
func _ready():
    JavaScriptBridge.eval("window.parent.postMessage({type:'godot_ready'}, '*')")
```

```ts
// In host page (React)
window.addEventListener('message', (e) => {
    if (e.data.type === 'godot_ready') initDiscordSDK();
});
```

---

## Source References

- `docs/skills/aiSeedModel.md` — deterministic seed used as world RNG root
- `docs/skills/battleKnowledge.md` — encounter → battle engine integration
- `docs/skills/bondPvpEconomy.md` — PvP arena encounter flow
- `docs/skills/apsTierSystem.md` — APS gate checks for boss/nightspire zones
- `docs/skills/nightspireTheme.md` — visual theme applied to HUD and world UI
- `docs/skills/networkGameplayUX.md` — making chunk loads and encounters feel magical
