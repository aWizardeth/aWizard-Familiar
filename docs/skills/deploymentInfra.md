# Skill: Deployment & Infrastructure

> aWizard's knowledge of hosting topology, CI/CD pipelines, environment configuration, and the boundary between Vercel, VPS, and blockchain nodes.

---

## Domain

aWizard can guide:

- **Vercel deployment** — static Vite builds, serverless functions, CDN
- **VPS deployment** — PM2/Docker for bots and servers, reverse proxy
- **Environment management** — secret separation, VITE_ prefix rules
- **Discord Developer Portal** — Activity URL mappings, OAuth2 config
- **CI/CD** — GitHub Actions for build + deploy on push
- **Operational support tooling** — database provisioning, wheel publishing, package release hygiene

## Hosting Topology

```
Vercel (Static + Serverless)
  |- awizard-gui (Vite build)
  |- /api/token (OAuth2 exchange)
  \- /api/nft/gate (NFT verification)

VPS (Persistent Processes)
  |- aWizard Discord Bot (Discord.js, WebSocket)
  |- gym-server (Express, SQLite, port 3001)
  \- Chia node (blockchain daemon)

Vercel (Separate Project)
  \- bow-app (Next.js, port 3000)
```

## Deployment Pipeline

```
git push main
  |--> Vercel auto-deploys awizard-gui
  |      Static assets -> CDN
  |      Serverless functions -> edge
  \--> (Separate) VPS deploy via PM2/Docker
         aWizard bot -> persistent WebSocket
         gym-server -> Express + SQLite
```

## Secret Separation Rules

| Prefix    | Accessible from | Safe for          |
| --------- | --------------- | ----------------- |
| `VITE_`   | Client bundle   | Public values only |
| No prefix | Server only     | Secrets, keys      |

**NEVER** put `DISCORD_CLIENT_SECRET` in a `VITE_` variable.

## Operational Toolchain

### Database Manager

Chia's `database-manager` is a useful reference for declarative infra operations:
- Config-driven user and database provisioning via YAML
- `validate` command to catch config errors before applying
- `apply` command to create databases, users, and grants
- `ENV:` value expansion for secrets and deployment-time injection
- Safe default network restriction behavior that falls back to `localhost`

This is the right pattern for future shared infra services when quest systems,
leaderboards, or back-office tools need managed database access.

### Build Wheels

`build-wheels` is a release-engineering reference for Python package maintenance:
- focused on automating wheel builds for `pypi.chia.net`
- useful when internal Chia tooling needs repeatable binary/package publishing
- relevant for Windows support, packaging consistency, and CI-based artifact generation

This matters if any of our Chia helper scripts graduate into reusable internal tools.

### Python Tooling Expectations

The current Chia reference tools commonly:
- pin specific `chia-blockchain` versions
- assume isolated virtual environments
- rely on wallet or full-node RPC availability
- are safer to automate through explicit CLI phases than ad hoc scripts

## Reference Repos

| Repo | Pattern Integrated |
| ---- | ------------------- |
| chia-gaming | Mini-Eltoo state channels, potato protocol |
| ChiaRPSGame | 3-party server signing, SpendBundle construction |
| sage | WalletConnect CHIP-0002 commands |
| secure-the-mint | NFT pre-launcher + eve spend pattern |
| chia-gaming-tracker | Room discovery, state channel tracking |
| rue | CLVM language for contract authoring |
| chia-wallet-sdk | BLS AggSig, CoinSpend driver |
| database-manager | Declarative database provisioning |
| build-wheels | Python wheel release automation |

## Source References
- `arcane-battle-protocol/DEPLOYMENT.md` — full deployment guide
- `awizard-gui/docs/ARCHITECTURE.md` — hosting and auth design
- `https://github.com/Chia-Network/database-manager` — config-driven database ops
- `https://github.com/Chia-Network/build-wheels` — package build automation
