# Skill: Tournament System

> aWizard's knowledge of tournament brackets, APS-seeded matchmaking, prize pools, and championship flows.

---

## Domain

aWizard can explain and manage:

- **Tournament creation** — announcement, entry deadlines, prize pools
- **Registration** — wallet + current APS/tier from NFT
- **Bracket seeding** — APS-ranked matchups (strongest vs weakest in round 1)
- **Match orchestration** — PvP Agent runs each bout
- **Prize distribution** — token or NFT rewards to champion
- **Stats persistence** — all results recorded on-chain

## Tournament Flow

```
1. Tournament announced (entry deadline + prize pool)
2. Players register (wallet + APS from NFT)
3. APS-seeded bracket generated
4. For each round:
   a. Schedule matches
   b. PvP Agent runs each match
   c. APS updated after each match
   d. Winners advance, losers eliminated
5. Final match -> champion determined
6. Prize distributed
7. Final standings published to leaderboard
```

## Bracket Seeding

```
Rank 1 vs Rank N
Rank 2 vs Rank N-1
Rank 3 vs Rank N-2
...
```

Highest APS faces lowest APS in round 1 — ensures competitive balance across rounds.

## Skills Chained
- `pvp_agent` — runs each bracket match
- `aps_calculator` — tier and APS tracking
- `leaderboards` — final standings
- `bond_handler` — prize pool management

## Source References
- `arcane-battle-protocol/tournament/tournament_agent.md` — full implementation
