# Skill: Network Gameplay UX

> Making every API call, blockchain transaction, and WebSocket event feel like a spell being cast
> — not a spinner on a grey background.
>
> This skill governs how the aWizard frontend turns network I/O into immersive, thematic interactions
> using the Nightspire design system.

---

## Domain

aWizard can:

- **Design spell-cast UX** — map network operations to in-world metaphors
- **Build loading states** that feel alive (shimmer, glow pulse, arcane runes animating)
- **Wire optimistic UI** for blockchain operations so players never stare at a dead screen
- **Scaffold real-time WebSocket patterns** for live battle updates
- **Handle errors as "curses"** with recovery paths built into the UI
- **Produce typed network hooks** (`useSpellCast`, `useChunkLoader`, `useGymBattle`) that abstract fetch/WS

---

## Core Philosophy

> Every network call is a spell. It has an incantation (request), a cast animation (pending),
> an effect (success), and a misfire (error). Never show raw HTTP status codes to the player.

| Network State | In-World Metaphor | UI Treatment |
|---------------|-------------------|--------------|
| `idle` | Wizard at rest | Normal UI — glow border dim |
| `pending` | Spell incantating | Glow pulse animation, rune spinner, thematic copy |
| `success` | Spell lands | Flash burst, success glow (green `--success`), loot reveal or state update |
| `error` | Curse / backfire | Red glow pulse (`--danger`), curse message, retry rune |
| `stale` | Fading enchantment | Muted glow, "Reconnecting…" shimmer |

---

## The Spell-Cast Hook Pattern

A single typed hook wraps every mutating network operation:

```ts
// src/hooks/useSpellCast.ts
import { useState } from 'react';

type SpellState = 'idle' | 'casting' | 'success' | 'cursed';

interface SpellCastResult<T> {
  state: SpellState;
  data: T | null;
  error: string | null;
  cast: (...args: unknown[]) => Promise<void>;
}

export function useSpellCast<T>(
  spellFn: (...args: unknown[]) => Promise<T>,
  opts?: { onSuccess?: (data: T) => void; onCurse?: (err: Error) => void }
): SpellCastResult<T> {
  const [state, setState] = useState<SpellState>('idle');
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<string | null>(null);

  const cast = async (...args: unknown[]) => {
    setState('casting');
    setError(null);
    try {
      const result = await spellFn(...args);
      setData(result);
      setState('success');
      opts?.onSuccess?.(result);
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Unknown curse';
      setError(message);
      setState('cursed');
      opts?.onCurse?.(err instanceof Error ? err : new Error(message));
    }
  };

  return { state, data, error, cast };
}
```

**Usage:**
```tsx
const { state, cast } = useSpellCast(gymClient.challengeGym);

<SpellButton state={state} onClick={() => cast(gymId)}>
  ⚔️ Challenge Gym
</SpellButton>
```

---

## SpellButton Component

```tsx
// src/components/SpellButton.tsx
import type { SpellState } from '../hooks/useSpellCast';

const LABELS: Record<SpellState, string> = {
  idle:    '',        // caller provides
  casting: 'Casting…',
  success: 'Spell Cast!',
  cursed:  'Cursed — Retry?',
};

const GLOW: Record<SpellState, string> = {
  idle:    'border-[var(--border-color)]',
  casting: 'border-[var(--accent)] animate-pulse',
  success: 'border-[var(--success)] shadow-[0_0_18px_var(--success)]',
  cursed:  'border-[var(--danger)]  shadow-[0_0_18px_var(--danger)]',
};

export function SpellButton({
  state, children, onClick, disabled,
}: { state: SpellState; children: React.ReactNode; onClick: () => void; disabled?: boolean }) {
  return (
    <button
      onClick={onClick}
      disabled={disabled || state === 'casting'}
      className={`px-4 py-2 rounded border-2 transition-all duration-300 font-semibold
        text-[var(--text-primary)] bg-[var(--bg-card)] ${GLOW[state]}`}
    >
      {state !== 'idle' ? LABELS[state] : children}
    </button>
  );
}
```

---

## Loading States — Arcane Shimmer

Never use a generic grey skeleton. Use the Nightspire glow palette:

```tsx
// src/components/ArcaneLoader.tsx
export function ArcaneLoader({ label = 'Summoning…' }: { label?: string }) {
  return (
    <div className="flex flex-col items-center gap-3 py-8">
      {/* Spinning rune ring */}
      <div className="w-12 h-12 rounded-full border-2 border-[var(--border-color)]
        border-t-[var(--glow-inner)] animate-spin" />
      <p className="text-[var(--text-muted)] text-sm animate-pulse">{label}</p>
    </div>
  );
}
```

**Thematic labels by operation:**

| Operation | Loader Label |
|-----------|-------------|
| Chunk load | `"Scrying the next region…"` |
| Gym connect | `"Contacting the Gym Master…"` |
| Blockchain tx submit | `"Inscribing the spell on-chain…"` |
| NFT fetch | `"Summoning your familiar…"` |
| Leaderboard fetch | `"Reading the Hall of Echoes…"` |
| OAuth token exchange | `"Verifying your arcane seal…"` |
| Battle turn submit | `"Channeling your spell…"` |

---

## Gym Server Client (`gymClient.ts`)

```ts
// src/lib/gymClient.ts
const GYM_BASE = import.meta.env.VITE_GYM_SERVER_URL ?? 'http://localhost:3001';

async function apiFetch<T>(path: string, options?: RequestInit): Promise<T> {
  const res = await fetch(`${GYM_BASE}${path}`, {
    ...options,
    headers: { 'Content-Type': 'application/json', ...options?.headers },
  });
  if (!res.ok) {
    const body = await res.text().catch(() => res.statusText);
    throw new Error(`[aWizard] Gym curse (${res.status}): ${body}`);
  }
  return res.json() as Promise<T>;
}

export const gymClient = {
  challengeGym: (gymId: string, playerWallet: string) =>
    apiFetch<{ channelId: string }>('/gym/challenge', {
      method: 'POST',
      body: JSON.stringify({ gymId, playerWallet }),
    }),

  submitTurn: (channelId: string, action: string, signature: string) =>
    apiFetch<{ state: BattleState }>('/gym/turn', {
      method: 'POST',
      body: JSON.stringify({ channelId, action, signature }),
    }),

  closeChannel: (channelId: string, finalState: BattleState) =>
    apiFetch<{ txId: string }>('/gym/close', {
      method: 'POST',
      body: JSON.stringify({ channelId, finalState }),
    }),
};
```

---

## Tracker Client (Redis Lobby)

```ts
// src/lib/trackerClient.ts
const TRACKER_BASE = '/api/tracker';

export const trackerClient = {
  createLobby: (playerId: string, wagerId: number) =>
    fetch(`${TRACKER_BASE}/lobbies`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ playerId, wagerId }),
    }).then(r => r.json()),

  joinLobby: (lobbyId: string, playerId: string) =>
    fetch(`${TRACKER_BASE}/lobbies/${lobbyId}/join`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ playerId }),
    }).then(r => r.json()),

  pollLobby: (lobbyId: string) =>
    fetch(`${TRACKER_BASE}/lobbies/${lobbyId}`).then(r => r.json()),
};
```

---

## Real-Time Battle (WebSocket)

For live PvP turn updates, use a persistent WebSocket with automatic reconnect:

```ts
// src/lib/battleSocket.ts
export function createBattleSocket(channelId: string, onMessage: (msg: BattleMessage) => void) {
  const WS_URL = import.meta.env.VITE_GYM_WS_URL ?? 'ws://localhost:3001';
  let ws: WebSocket;
  let reconnectTimer: ReturnType<typeof setTimeout>;

  function connect() {
    ws = new WebSocket(`${WS_URL}/battle/${channelId}`);

    ws.onmessage = (e) => {
      try {
        onMessage(JSON.parse(e.data) as BattleMessage);
      } catch {
        console.error('[aWizard] Cursed socket message — parse failed');
      }
    };

    ws.onclose = () => {
      reconnectTimer = setTimeout(connect, 2000); // reconnect after 2s
    };
  }

  connect();

  return {
    send: (msg: BattleAction) => ws.readyState === WebSocket.OPEN && ws.send(JSON.stringify(msg)),
    close: () => { clearTimeout(reconnectTimer); ws.close(); },
  };
}
```

**React hook wrapping the socket:**
```ts
// src/hooks/useBattleSocket.ts
import { useEffect, useRef } from 'react';
import { createBattleSocket } from '../lib/battleSocket';
import { useBattleStore } from '../store/battleStore';

export function useBattleSocket(channelId: string | null) {
  const socketRef = useRef<ReturnType<typeof createBattleSocket> | null>(null);
  const applyMessage = useBattleStore(s => s.actions.applyMessage);

  useEffect(() => {
    if (!channelId) return;
    socketRef.current = createBattleSocket(channelId, applyMessage);
    return () => socketRef.current?.close();
  }, [channelId, applyMessage]);

  return { sendAction: (action: BattleAction) => socketRef.current?.send(action) };
}
```

---

## Blockchain Transaction UX (Chia / WalletConnect)

Blockchain writes have 3–60 second confirmation windows. Make this magical, not painful:

```
Player clicks "Submit Bond" or "Settle Battle"
  ↓
STATE: casting  → "Inscribing the spell on-chain…" + rune spinner
  ↓
Tx submitted to mempool
  ↓
STATE: pending  → "Awaiting confirmation from the Chia weave…" + block-counter progress bar
  ↓
Tx confirmed (poll tx status via /api/chain/tx/:id every 4s)
  ↓
STATE: success  → burst animation + "Spell sealed on-chain ✨" + show tx explorer link
```

**Progress bar component:**
```tsx
// src/components/ChainProgressBar.tsx
export function ChainProgressBar({ confirmations, required = 1 }: {
  confirmations: number; required?: number;
}) {
  const pct = Math.min((confirmations / required) * 100, 100);
  return (
    <div className="w-full h-2 rounded bg-[var(--bg-tertiary)] overflow-hidden">
      <div
        className="h-full bg-[var(--accent)] transition-all duration-500
          shadow-[0_0_8px_var(--accent)]"
        style={{ width: `${pct}%` }}
      />
    </div>
  );
}
```

---

## Error Handling as Curses

Never show raw error messages. Map known errors to thematic copy:

```ts
// src/lib/curseMessages.ts
const CURSE_MAP: Record<string, string> = {
  'Network request failed':   'The arcane network is silent. Check your connection.',
  '401':                      'Your seal has expired. Re-authenticate with Discord.',
  '403':                      'Access denied by the guild enchantment.',
  '404':                      'That spell does not exist in this realm.',
  '429':                      'Too many incantations. Wait a moment before retrying.',
  '500':                      'The Gym Master\'s tower is dark. Try again shortly.',
  'WalletNotFound':           'No Sage wallet detected. Install the Sage extension first.',
  'UserRejected':             'Spell cancelled — you withdrew the incantation.',
  'InsufficientFunds':        'Not enough XCH to stake this bond.',
};

export function cursify(raw: string): string {
  for (const [key, message] of Object.entries(CURSE_MAP)) {
    if (raw.includes(key)) return message;
  }
  return `Unknown curse: ${raw}`;
}
```

**Usage:**
```tsx
{state === 'cursed' && (
  <p className="text-[var(--danger)] text-sm mt-2 animate-pulse">
    🔴 {cursify(error ?? '')}
  </p>
)}
```

---

## Polling Hook (Lobby / TX Status)

```ts
// src/hooks/useOracle.ts  — periodic polling with thematic label
import { useEffect, useState } from 'react';

export function useOracle<T>(
  fetch: () => Promise<T>,
  intervalMs: number,
  enabled = true,
) {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    if (!enabled) return;
    let active = true;

    const poll = async () => {
      try {
        const result = await fetch();
        if (active) setData(result);
      } catch (e) {
        if (active) setError(e instanceof Error ? e.message : 'Oracle silent');
      }
    };

    poll();
    const id = setInterval(poll, intervalMs);
    return () => { active = false; clearInterval(id); };
  }, [fetch, intervalMs, enabled]);

  return { data, error };
}
```

---

## World Chunk Loading UX

When the player walks near a chunk edge, preload the adjacent chunks silently. If a chunk
fetch fails, show a "fog of war" tile overlay rather than crashing the overworld:

```ts
// src/store/worldStore.ts (chunk loading action)
loadChunk: async (x, y) => {
  const key = `chunk_${x}_${y}`;
  if (get().loadedChunks[key]) return; // already cached

  try {
    const chunk = await fetch(`/api/world/chunk?x=${x}&y=${y}`).then(r => {
      if (!r.ok) throw new Error(`${r.status}`);
      return r.json();
    });
    set(s => ({ loadedChunks: { ...s.loadedChunks, [key]: chunk } }));
  } catch {
    // Silently mark as "fog" — player sees dark tiles, not a crash
    set(s => ({ loadedChunks: { ...s.loadedChunks, [key]: { id: key, fog: true } } }));
  }
}
```

---

## Discord Activity Event Bridge

Wrap Discord SDK events with the same spell-cast pattern so UI remains consistent
even when the SDK fires or the participant list changes:

```ts
// src/lib/discordBridge.ts
import { DiscordSDK } from '@discord/embedded-app-sdk';

export async function castDiscordSpell<T>(
  sdk: DiscordSDK,
  spellName: string,
  fn: (sdk: DiscordSDK) => Promise<T>,
): Promise<T> {
  try {
    return await fn(sdk);
  } catch (err) {
    throw new Error(`[aWizard] Discord spell "${spellName}" backfired: ${
      err instanceof Error ? err.message : err
    }`);
  }
}

// Usage:
const auth = await castDiscordSpell(sdk, 'authorize', (s) =>
  s.commands.authorize({ client_id: VITE_DISCORD_CLIENT_ID, ... })
);
```

---

## Quick Reference — Hook × Operation Map

| Quest | Hook / Client | Key State Labels |
|-------|--------------|-----------------|
| Challenge gym | `useSpellCast(gymClient.challengeGym)` | Casting → "Contacting Gym Master…" |
| Submit turn | `useSpellCast(gymClient.submitTurn)` | Casting → "Channeling spell…" |
| Create PvP lobby | `useSpellCast(trackerClient.createLobby)` | Casting → "Opening the dueling circle…" |
| Poll lobby | `useOracle(trackerClient.pollLobby, 2000)` | Polling every 2s silently |
| Load world chunk | `worldStore.loadChunk()` | Fog-of-war fallback on error |
| Submit bond on-chain | `useSpellCast(chiaClient.submitBond)` | Pending → `ChainProgressBar` |
| Fetch NFT | `useSpellCast(nftClient.fetchNFT)` | Casting → "Summoning familiar…" |
| Discord auth | `castDiscordSpell(sdk, 'authorize', …)` | Pending → "Verifying arcane seal…" |

---

## Source References

- `docs/skills/nightspireTheme.md` — CSS tokens for all glow/state colours
- `docs/skills/battleKnowledge.md` — battle flow driving most spell casts
- `docs/skills/discordActivityAuth.md` — OAuth2 token exchange spell
- `docs/skills/bondPvpEconomy.md` — on-chain bond submission UX
- `docs/skills/snesWorldEngine.md` — world chunk loading network patterns
