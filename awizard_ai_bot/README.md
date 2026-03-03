# aWizard AI Bot 🧙

> OpenAI-powered Discord assistant for the Arcane BOW ecosystem.  
> Runs on a VPS alongside the main `awizard_bot`. Communicates with it via shared JSON files on disk.

---

## What it does

| Feature | Detail |
|---------|--------|
| **AI conversation** | GPT-4o wizard persona with Arcane BOW domain knowledge |
| **Wallet / holdings lookup** | Reads from `awizard_bot/master_list.json` and `wallet_list/` to give personalised replies about XCH holdings, NFTs, and token power |
| **Per-user workspace** | Each Discord user gets a sandboxed `data/users/<id>/` folder — save notes, strategy docs, etc. |
| **MCP server** | Exposes tools over Model Context Protocol (stdio transport) for external agent use |
| **Security** | Path-traversal protection, 4 KB file cap, 50 KB notes cap, HTML/shell injection sanitisation |

---

## Architecture

```
awizard_ai_bot/
├── wizard_ai.js          ← Main process: Discord.js client + OpenAI chat loop + MCP server
├── package.json
├── .env                  ← DISCORD_TOKEN, OPENAI_API_KEY, DISCORD_CHANNEL_ID
└── data/
    └── users/
        └── <discord_id>/
            ├── notes.md  ← append-only personal notes
            └── *.md      ← user-saved strategy/reference files (max 20)

# Shared read-only data (written by awizard_bot):
../awizard_bot/
├── master_list.json          ← { discordId: { username, spellPower, … } }
└── wallet_list/<discordId>.json  ← { address, cats, collections, … }
```

---

## Environment Variables

```env
# Required
DISCORD_TOKEN=your_discord_bot_token
OPENAI_API_KEY=sk-...
DISCORD_CHANNEL_ID=channel_id_to_listen_in   # bot only responds in this channel

# Optional
OPENAI_MODEL=gpt-4o          # default: gpt-4o
MAX_HISTORY=20               # conversation turns to keep in context
```

---

## Running

```bash
npm install
cp .env.example .env   # fill in values
npm start
```

Runs as a persistent Node process. Use `pm2` or `systemd` on a VPS to keep it alive:

```bash
pm2 start wizard_ai.js --name awizard-ai-bot
pm2 save
```

---

## MCP Tools (exposed via stdio)

| Tool | Description |
|------|-------------|
| `chat` | Send a message to the aWizard AI, returns a reply |
| `save_note` | Append a note to the user's `notes.md` workspace file |
| `save_file` | Save a named markdown file to the user's workspace |
| `read_file` | Read a file from the user's workspace |
| `list_files` | List files in the user's workspace |
| `lookup_wallet` | Look up Chia wallet holdings and NFT collections for a Discord user |

---

## Relationship to Other Services

| Service | Relationship |
|---------|-------------|
| `awizard_bot` | Sibling process — aWizard AI reads its `master_list.json` and `wallet_list/` (read-only) |
| `The-Nightspire` (`awizard-gui`) | Activity GUI calls `VITE_AWIZARD_BOT_URL` REST endpoint (TODO: expose `/chat` route) |
| `gym-server` | Independent — battle results flow through tracker, not directly to AI bot |
| `bow-app` | Independent — web client; user links wallet there separately |

---

## Status

- **v1.1.0** (2026-03-02) — MCP server, wallet lookup, per-user workspace, OpenAI GPT-4o chat
- Pending: expose `/chat` HTTP endpoint for The Nightspire Activity to consume
