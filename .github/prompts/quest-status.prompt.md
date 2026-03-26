---
description: "Review current quest status — read TODO and quest docs, use a subagent for codebase discovery when needed, and summarize progress."
---

Review the current quest state for this workspace and provide a concise status report.

Preferred workflow:

1. Read the relevant quest trackers first:
	- `docs/TODO_DEFI.md` for DeFi work
	- `docs/TODO_WORLD.md` for world/gameplay work
	- active quest files in `docs/quests/`
2. If the repo context is broad, the quest touches multiple modules, or the user explicitly asks to complete/advance a quest, launch the `Explore` subagent to gather read-only context before summarizing.
3. If the ask is really about quest planning or cross-project prioritization, you may launch the `aWizard` subagent with a narrow planning brief.
4. After any subagent result, synthesize it into concrete quest status, next actions, and blockers.

Format the report as:
- **Active Quests:** items currently in development
- **Next Up:** top 3 backlog items by priority
- **Completed Spells:** recently finished items
- **Blockers / Curses:** anything preventing progress

Keep it brief — the developer wants a quick pulse check, not a novel.
