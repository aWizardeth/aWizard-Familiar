# Skill: APS & Tier System

> aWizard's mastery of the Arcane Power Score (APS) formula, tier thresholds, ability unlocks, and progression mechanics.

---

## Domain

aWizard can calculate, explain, and predict:

- **APS scoring** — wins, losses, and point values
- **Tier progression** — thresholds and downgrades
- **Ability unlocks** — what each tier grants
- **Tier evaluation** — when upgrades/downgrades trigger

## Formula

```
APS = (Wins x 3) - (Losses x 2)
```

Minimum APS is `0` — cannot go negative, but tier can drop as APS decreases.

## Tier Thresholds

| Tier          | APS Required | Ability Unlocked                          |
| ------------- | ------------ | ----------------------------------------- |
| Initiate      | >= 0         | —                                         |
| Adept         | >= 10        | —                                         |
| Archmage      | >= 25        | `teleport: true` → enables `/teleport_bow` |
| Grand Wizard  | >= 50        | `spell_boost += 1`                        |

Tier **can downgrade** if APS falls below threshold.

## Quick Reference Examples

| Wins | Losses | APS | Tier         |
| ---- | ------ | --- | ------------ |
| 4    | 1      | 10  | Adept        |
| 10   | 2      | 26  | Archmage     |
| 20   | 5      | 50  | Grand Wizard |
| 5    | 7      | 1   | Initiate     |

## Source References
- `arcane-battle-protocol/gyms/arcane-bow/aps_calculator.md` — full implementation
- `arcane-battle-protocol/ARCHITECTURE.md` — tier/APS section
