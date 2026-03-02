# Skill: AI Seed Model

> aWizard's understanding of deterministic AI move generation — seed construction, RNG seeding, difficulty scaling, and move selection.

---

## Domain

aWizard can explain and debug:

- **Seed generation** — SHA-256 from battle inputs
- **Deterministic RNG** — why identical inputs produce identical battles
- **Difficulty scaling** — how AI parameters change per tier
- **Move selection** — weighted choices, prediction bonuses, counter logic
- **State-channel verification** — why determinism is required

## Seed Generation

```
Input:  player_wallet + gym_id + tier_attempted + battle_counter
Method: SHA-256 -> integer -> random.seed()
```

Same inputs = same AI behavior every time. Essential for state-channel dispute resolution.

## AI Parameters by Tier

| Tier         | Aggressiveness | Skill Usage | Predict Moves | Recovery Use |
| ------------ | -------------- | ----------- | ------------- | ------------ |
| Initiate     | 0.60           | 0.50        | 0.20          | 0.30         |
| Adept        | 0.70           | 0.65        | 0.40          | 0.50         |
| Archmage     | 0.80           | 0.80        | 0.60          | 0.70         |
| Grand Wizard | 0.90           | 0.95        | 0.80          | 0.90         |

## Move Selection

Available moves: `attack`, `defend`, `spell`, `recover`

Weights derived from AI parameters -> normalized -> `random.choices()` with seeded RNG.

**Prediction bonus:** If AI correctly predicts player's next move (based on their last move), counter-move gets 1.25x modifier.

## Source References
- `arcane-battle-protocol/gyms/arcane-bow/ai_seed_model.md` — full implementation
- `gym-server/src/ai.ts` — TypeScript port
