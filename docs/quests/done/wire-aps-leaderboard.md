# Quest: Wire APS Leaderboard & NFT Upgrade — `bow-app`

> **Zero overlap guarantee:**
> - `bootstrap-aggregator-dex.md` → `projects/chia-aggregator/` ✅ separate
> - `bootstrap-portal-arbitrage.md` → `projects/chia-portal/` ✅ separate
> - `build-chia-multisig-interface.md` → new multisig project ✅ separate
> - **This quest** → `projects/bow-app/app/` only

---

## Objective

The APS formula, tier system, and leaderboard are fully spec'd in the protocol but **never wired into the game client website**. Wire them in three targeted changes:

1. **`lib/aps.ts`** — APS calculator + tier resolver (new utility)
2. **`/nft` page upgrade** — use full `MagicBOWNFT` schema: show APS, tier name, abilities
3. **`/leaderboard` route** — new page ranking all known wallets by APS (reads from tracker)
4. **`/stats` page** — add APS score + tier badge to the existing stats summary

---

## APS Formula (from `apsTierSystem.md`)

```
APS = (Wins × 3) − (Losses × 2),  floor: 0
```

| Tier | APS | Ability |
|------|-----|---------|
| Initiate | ≥ 0 | — |
| Adept | ≥ 10 | — |
| Archmage | ≥ 25 | `teleport: true` |
| Grand Wizard | ≥ 50 | `spell_boost += 1` |

---

## Step 1 — `app/lib/aps.ts` (new file)

Pure utility, no side effects, no imports except the types file:

```typescript
export type TierName = 'Initiate' | 'Adept' | 'Archmage' | 'Grand Wizard';

export interface ApsResult {
  aps: number;
  tier: TierName;
  teleport: boolean;
  spellBoost: number;
}

export function calcAps(wins: number, losses: number): ApsResult {
  const aps = Math.max(0, wins * 3 - losses * 2);
  const tier: TierName =
    aps >= 50 ? 'Grand Wizard' :
    aps >= 25 ? 'Archmage'     :
    aps >= 10 ? 'Adept'        : 'Initiate';
  return {
    aps,
    tier,
    teleport: aps >= 25,
    spellBoost: aps >= 50 ? 1 : 0,
  };
}

export const TIER_CONFIG: Record<TierName, { emoji: string; color: string; bgClass: string }> = {
  'Initiate':    { emoji: '🧙',    color: '#94a3b8', bgClass: 'from-slate-400 to-slate-600' },
  'Adept':       { emoji: '🧙‍♂️✨',  color: '#60a5fa', bgClass: 'from-blue-400 to-blue-700' },
  'Archmage':    { emoji: '🔮',    color: '#a78bfa', bgClass: 'from-purple-400 to-purple-700' },
  'Grand Wizard':{ emoji: '👑🔮',  color: '#fbbf24', bgClass: 'from-yellow-400 to-yellow-600' },
};
```

---

## Step 2 — `/nft` page upgrade (`app/nft/page.tsx`)

The current page uses a simplified `NftRecord` type (tier: 1|2|3, no APS). Replace with:

```typescript
interface NftRecord {
  nftId: string;
  launcherId: string;
  wins: number;
  losses: number;
  aps: number;
  tier: TierName;
  gymName: string;
  mintedAt: number;
  soulbound: true;
  unlockedAbilities: { teleport: boolean; spellBoost: number };
}
```

Update mock data to use real `calcAps()` values.

**Card UI additions:**
- Tier name badge (colour-coded per `TIER_CONFIG`)
- APS number with glow (e.g. `⚡ 26 APS`)
- W/L record: `12W / 3L`
- Ability chips: `⚡ Teleport` (if unlocked) | `🌀 Spell Boost ×N` (if > 0)
- `🔒 Soulbound` pill at bottom of card
- `calcAps(wins, losses)` drives all derived fields — no hardcoded tier numbers

---

## Step 3 — `/leaderboard` page (new: `app/leaderboard/page.tsx`)

New page showing all tracked wallets ranked by APS:

```
/leaderboard
```

**Data source:** `trackerClient.getTopPlayers()` — a new method to add to `lib/trackerClient.ts`
that reads all known wallet stats from Upstash and returns them sorted by APS.
For this quest: mock with 10 seeded entries using `calcAps()`.

**UI:**
- Table: Rank | Fingerprint (short) | Tier badge | APS | W | L | Win%
- Tier badge colours from `TIER_CONFIG`
- Gold border on rank 1 row; subtle glow on top 3
- "Your rank" row highlighted if wallet is connected (match by fingerprint)
- "Data sourced from on-chain battle records — verifiable, no central DB" footnote
- Add `/leaderboard` to `NavBar.tsx` link list

---

## Step 4 — `/stats` page APS banner

The existing `/stats` page shows wins/losses but no APS. Add above the battle history:

- `APS: <number>` with the tier badge inline
- e.g. `⚡ 26 APS  🔮 Archmage`
- Tier upgrade progress bar: `26 / 50 to Grand Wizard`
- Uses `calcAps(stats.wins, stats.losses)` directly from the already-loaded `BattleStats`

---

## Success Criteria

| Criterion | Target |
|-----------|--------|
| `npx tsc --noEmit` | 0 errors |
| `npm run build` | clean Next.js build |
| `lib/aps.ts` | `calcAps()` + `TIER_CONFIG`, zero `any`, pure functions |
| `/nft` page | Full schema, APS glow number, tier name, ability chips |
| `/leaderboard` | 10-row table, tier badges, highlights connected wallet |
| `/stats` | APS banner + tier progress bar above battle history |
| NavBar | `/leaderboard` link added |

---

## Parallel Safety

| Active Quest | Codebase | Overlap |
|---|---|---|
| `bootstrap-aggregator-dex` | `projects/chia-aggregator/` | ❌ none |
| `bootstrap-portal-arbitrage` | `projects/chia-portal/` | ❌ none |
| `build-chia-multisig-interface` | new multisig project | ❌ none |
| **This quest** | `projects/bow-app/app/` | ✅ isolated |
