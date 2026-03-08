# TODO — SNES World Engine & Gameplay UX

> Quest log for the Arcane BOW overworld — split-track development.
>
> 🧙 **Frontend Wizard** — builds the React/Phaser UI inside `awizard-gui`
> ⚙️ **Backend Wizard** — builds the world API, chunk server, NPC/quest engine
>
> Quests marked **[FE]** are front-end-only and unblock immediately.
> Quests marked **[BE]** are back-end-only.
> Quests marked **[SYNC]** need both sides before going live.
>
> Start with `docs/skills/README.md` for the skill index, then open `snesWorldEngine.md`, `networkGameplayUX.md`, and `nightspireTheme.md`.
> Use `docs/ARCHITECTURE_INDEX.md` first if the work needs product or system-level architecture context.

---

## 🗺️ Phase 0 — Foundation (No Blockers)

Both wizards lay groundwork independently — nothing depends on the other side yet.

### [FE] Project Scaffold ✅ COMPLETE (advanced implementation discovered)
- [x] Add `src/store/worldStore.ts` — ✅ **COMPLETE** Advanced Zustand slice with movement, chunk loading, battle integration  
- [x] Add `src/lib/worldTypes.ts` — ✅ **COMPLETE** Complete type system (ChunkData, BiomeId, NpcEntry, QuestData, EncounterData, BattleResult)
- [x] Add `src/lib/rng.ts` — ✅ **COMPLETE** Mulberry32 seeded PRNG with chunk seeding + utility functions
- [x] Add `VITE_WORLD_API_URL` to `.env.example` — ✅ **COMPLETE** (already present)
- [x] Add `VITE_GYM_SERVER_URL` to `.env.example` — ✅ **COMPLETE** (already present) 
- [x] **CREATED** `src/lib/worldClient.ts` — ✅ Real world API client with graceful fallback to procedural generation

### [BE] API Scaffold
- [ ] Bootstrap `world-server/` project (Express + TypeScript, port 3002)
- [ ] Define chunk storage schema — `chunks/chunk_x_y.json` file structure
- [ ] Seed initial chunks for `wizard_academy` starting zone (3×3 grid, `chunk_-1_-1` → `chunk_1_1`)
- [ ] Create biome registry file `world/registry.json` with `forest`, `desert`, `ruins`, `wizard_academy`
- [ ] `GET /world/chunk?x=:x&y=:y` → returns `ChunkData` JSON (file-based at first, DB later)
- [ ] `GET /world/biome/:id` → returns `BiomeData` JSON

---

## 🏗️ Phase 1 — Tile Renderer (FE-led, mocked data OK)

Frontend can use hardcoded chunk JSON from `snesWorldEngine.md` schemas until the API is ready.

### [FE] Tilemap Canvas Renderer ✅ COMPLETE
- [x] Install Phaser 3 (`phaser`) into `awizard-gui` ✅
- [x] Create `src/components/WorldCanvas.tsx` — Phaser game instance in a React ref container ✅
- [x] Load tileset texture from procedural generation — ✅ **ENHANCED** (graphics-based tile generation)
- [x] Render a 16×16 tile grid for one chunk from `ChunkData.tiles` 2D array ✅
- [x] Apply Nightspire background: `--bg-deep` as canvas clear colour, glow overlay ✅  
- [x] Scale tile size: 16px sprite × 3 = 48px display (crisp SNES feel) ✅

### [FE] Player Sprite & Movement
- [ ] Add player sprite to `public/sprites/wizard_player.png` (placeholder 16×16)
- [ ] Implement discrete grid movement: arrow keys + WASD, queued input (one tile per keydown)
- [ ] Animate: walking spritesheet, 4 directions, 3 frames each
- [ ] Sync player grid position to `worldStore.playerPos`
- [ ] Prevent movement into solid/wall tiles (read `ChunkData.tiles` collision layer)

### [FE] ArcaneLoader & SpellButton (no blockers)
- [ ] Create `src/components/ArcaneLoader.tsx` — spinning rune ring + thematic label prop
- [ ] Create `src/components/SpellButton.tsx` — 4-state glow button from `networkGameplayUX.md`
- [ ] Create `src/components/ChainProgressBar.tsx` — animated accent bar for tx confirmations
- [ ] Add `src/lib/curseMessages.ts` — `cursify()` error mapper (all known HTTP + wallet codes)

### [FE] useSpellCast Hook
- [ ] Create `src/hooks/useSpellCast.ts` — typed `SpellState` + `cast()` wrapper from skill doc

---

## 🌐 Phase 2 — Chunk Streaming (SYNC — needs BE chunk API)

Start wiring real data once `GET /world/chunk` endpoint is up.

### [BE] Chunk API Polish
- [ ] Return `404` with `{ fog: true }` when chunk does not exist (FE renders fog-of-war)
- [ ] Validate `x` and `y` query params are integers within world bounds (-100 to 100)
- [ ] Add CORS header allowing `localhost:5173` and the Activity domain
- [ ] Benchmark: chunk fetch must complete in < 50 ms (file read is fine at this stage)

### [FE] Dynamic Chunk Loading
- [ ] Wire `worldStore.loadChunk(x, y)` to `GET /world/chunk` via `src/lib/worldClient.ts`
- [ ] Preload radius: load all chunks within Manhattan distance 2 of player chunk
- [ ] Evict chunks outside distance 4 from `loadedChunks` to keep memory bounded
- [ ] Render fog-of-war dark overlay when `chunk.fog === true`
- [ ] Show `ArcaneLoader` label `"Scrying the next region…"` during first-load only

### [FE] Chunk Transition
- [ ] Detect player walking off chunk edge → trigger `loadChunk` for adjacent chunk
- [ ] Fade-to-black transition (200 ms) → teleport player to opposite edge of new chunk → fade in
- [ ] Play biome music on chunk enter (swap Phaser audio source when `biome` changes)

---

## 🧑 Phase 3 — NPCs & Dialogue (SYNC)

### [BE] NPC Engine
- [ ] Define `world/npcs/npc_manifest.json` with at least 3 NPCs in starting zone
- [ ] `GET /world/npc/:id` → returns `NpcEntry` (dialogue tree ref, quest id, sprite key)
- [ ] Store dialogue trees as `world/npcs/dialogues/:id.json` (ink-style simplified format from skill)

### [FE] NPC Rendering
- [ ] Render NPCs from current chunk's `npcs` array as 16×16 sprites on the tilemap
- [ ] Detect player adjacency (1 tile) + interaction key (E / Space) → open dialogue
- [ ] Create `src/components/DialogueBox.tsx` — Nightspire-styled text panel, typewriter effect
  - Text colour: `--text-primary`, background: `--bg-card`, border: `--border-color` glow
  - Choice buttons rendered as `SpellButton` with `idle` state
- [ ] Wire dialogue choices to next dialogue node (fetch from `NpcEntry.dialogue_tree`)
- [ ] On dialogue complete: trigger quest acceptance if `NpcEntry.quest_id` is set

### [FE] Quest HUD
- [ ] Create `src/components/QuestTracker.tsx` — collapsible panel, top-right corner
- [ ] Display active quest title + current step description
- [ ] Update from `worldStore.activeQuests` reactively

---

## ⚔️ Phase 4 — Encounter System (SYNC — needs gym-server + world-server)

### [BE] Encounter Trigger API
- [ ] `POST /world/encounter` with `{ chunkId, zone, playerWallet }` → server validates zone, rolls encounter rate, returns `{ triggered: bool, encounterData: EncounterData | null }`
- [ ] `EncounterData` includes `type`, `gym_id` (if gym_pve), `enemy_seed`, and `difficulty`
- [ ] Ensure rate is enforced server-side (client cannot force trigger)

### [FE] Encounter Flow
- [ ] On each player tile step: check if player is inside any `chunk.encounters[].zone` rect
- [ ] If in zone: call `POST /world/encounter` via `useSpellCast`
- [ ] If `triggered === true`: freeze overworld → dispatch `worldStore.triggerEncounter(data)`
- [ ] Fade to battle screen: `BattleTransition.tsx` — flash white → zoom rune circle → fade in battle UI
- [ ] Create `src/components/BattleTransition.tsx` — animated transition using `--glow-inner` + `--border-color`
- [ ] After battle resolves → `worldStore.resolveEncounter(result)` → fade back to overworld
- [ ] Apply battle result: update player APS (from `apsTierSystem.md`), inventory, quest steps

### [FE] Wild Encounter UI (lightweight, no state channel)
- [ ] `src/components/WildBattlePanel.tsx` — simplified panel for non-gym encounters
- [ ] Actions: Attack, Run, Inspect
- [ ] No bond staking, no state channel — single round resolve via `POST /gym/wild-turn`
- [ ] Show `ArcaneLoader` with label `"Channeling spell…"` during turn

---

## 🔴 Phase 5 — Real-Time Battle (SYNC — needs gym-server WS)

### [BE] WebSocket Battle (gym-server)
- [ ] Add `ws` package to `gym-server`
- [ ] Expose `ws://localhost:3001/battle/:channelId` endpoint
- [ ] Emit `BattleMessage` events: `turn_result`, `hp_update`, `battle_end`
- [ ] Authenticate connection with signed `channelId + playerWallet` token

### [FE] WebSocket Integration
- [ ] Create `src/lib/battleSocket.ts` — `createBattleSocket()` with auto-reconnect (from skill)
- [ ] Create `src/hooks/useBattleSocket.ts` — React wrapper, `sendAction()` exposed
- [ ] Wire into `src/components/GymBattlePanel.tsx` — replace polling with live WS events
- [ ] Show reconnecting state: `ArcaneLoader` with `"Reconnecting to the Gym…"` if socket drops
- [ ] HP bars animate smoothly on each `hp_update` message (`transition-width 400ms`)

---

## 🌍 Phase 6 — World Expansion (BE-led, FE supports)

### [BE] Procedural Chunk Generator
- [ ] `POST /world/generate-chunk?x=:x&y=:y` — admin-only endpoint, persists new chunk JSON
- [ ] Implement `biome_noise(x, y)` using simplex noise (seeded from world master seed)
- [ ] Implement `tile_generator(biome)` — weighted tile selection per biome
- [ ] Implement `encounter_placer()` — scatter encounter zones
- [ ] Implement `npc_placer()` — weighted NPC spawn
- [ ] Generate full `forest` biome ring (chunks `±2` from academy) as first expansion

### [FE] Minimap
- [ ] Create `src/components/Minimap.tsx` — 60×60 px canvas, top-left corner
- [ ] Render visited chunks as coloured squares (biome colour key from `nightspireTheme.md` palette)
- [ ] Blink current chunk. Show fog-of-war for unvisited neighbours
- [ ] Tap/click minimap → open full `WorldMap.tsx` modal

---

## 🏁 Completed

_(spells not yet cast — the scroll is blank)_

---

## 💡 Ideas Parking Lot

- **Dungeon portals** — special tile in ruins chunks that teleports to an instanced dungeon
- **PvP arena zones** — wizard_academy has a designated tile zone that opens the bond PvP lobby
- **World events** — server broadcasts `world_event` via WS (meteor shower, boss spawn) to all online players
- **Seasonal biomes** — biome tileset swaps for in-game calendar events
- **AI-generated pixel art** — Pixelorama pipeline feeds procedurally named sprites into the world sprite atlas
- **Boss tower** — nightspire biome, APS-gated (Tier 5+), auto-unlocks on leaderboard milestone
