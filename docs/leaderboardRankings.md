# Skill: Leaderboard & Rankings

> aWizard's ability to query, explain, and display APS-ranked leaderboards derived from on-chain NFT data.

---

## Domain

aWizard can:

- **Build leaderboards** — aggregate all NFT holders' APS
- **Rank players** — sort by APS with wins as tiebreaker
- **Query individual rank** — find a specific player's position
- **Explain methodology** — data comes from on-chain, verifiable, no central DB

## Data Source

Leaderboard data is derived from **on-chain NFT metadata** — all entries are verifiable and immutable. No centralized database required for rankings.

## Leaderboard Entry

```json
{
  "rank": 1,
  "wallet": "xch1top...",
  "aps": 73,
  "tier": "Grand Wizard",
  "wins": 26,
  "losses": 1,
  "win_rate": 96.3,
  "unlocked_abilities": { "teleport": true, "spell_boost": 1 }
}
```

## Ranking Algorithm

```
1. Fetch NFT metadata for all known holders
2. Calculate APS for each: (Wins x 3) - (Losses x 2)
3. Sort by APS descending, wins as tiebreaker
4. Assign rank 1..N
```

## Source References
- `arcane-battle-protocol/pvp/leaderboards.md` — full implementation
- `bow-app/app/stats/page.tsx` — stats page in game client
