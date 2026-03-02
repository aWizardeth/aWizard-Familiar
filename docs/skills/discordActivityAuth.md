# Skill: Discord Activity & Auth

> aWizard's expertise in Discord Embedded App SDK, OAuth2 flows, NFT-gated access, and Activity lifecycle management.

---

## Domain

aWizard can explain and implement:

- **Discord Embedded App SDK** — Activity lifecycle, ready/authorize/authenticate
- **OAuth2 token exchange** — code -> access_token via serverless function
- **NFT-gated access** — verify on-chain NFT ownership before granting entry
- **Role-gated access** — Discord server role checks as simpler alternative
- **Session management** — token in memory only, never persisted
- **CSP & security** — iframe origin allowlisting, rate limiting

## Auth Flows

### Discord OAuth2 (default)
```
Activity loads -> SDK.ready() -> SDK.commands.authorize()
-> returns code -> POST to /api/token (serverless)
-> exchange for access_token -> SDK.commands.authenticate()
-> user identity available in Zustand store
```

### NFT Gate (optional add-on)
```
After Discord auth -> user connects Chia wallet (WalletConnect)
-> call /api/nft/gate?wallet=<address>
-> server queries on-chain for approved collection NFTs
-> allowed: true/false
```

### Role Gate (simplest alternative)
```
After Discord auth -> check user's guild roles via Discord API
-> if user has "Wizard" role -> access granted
```

## Activity Frame Constraints

- Runs inside Discord's iframe — limited viewport
- Must handle both desktop and mobile layouts
- HTTPS required (Vercel provides this)
- CSP must allow Discord's iframe origins

## Required Environment Variables

| Variable                   | Scope    | Purpose                    |
| -------------------------- | -------- | -------------------------- |
| `DISCORD_CLIENT_ID`        | Server   | OAuth2 app ID              |
| `DISCORD_CLIENT_SECRET`    | Server   | Token exchange (never VITE_) |
| `VITE_DISCORD_CLIENT_ID`   | Client   | SDK initialization         |
| `VITE_REQUIRE_NFT_GATE`    | Client   | Enable/disable NFT check   |

## Source References
- `awizard-gui/src/discord.ts` — SDK bootstrap
- `awizard-gui/src/auth.ts` — auth utilities
- `awizard-gui/docs/ARCHITECTURE.md` — hosting and auth design
