---
description: "Review architecture decisions — hosting, auth, and deployment for the Discord Activity."
---

Review the current architecture documentation at `awizard-gui/docs/ARCHITECTURE.md` and answer questions about:

- **Hosting:** What goes on Vercel vs VPS? Is the current setup correct?
- **Auth flow:** Discord OAuth2 → token exchange → optional NFT gate
- **Environment variables:** Are all required vars documented?
- **Security:** Token handling, CSP headers, rate limiting
- **Deployment pipeline:** CI/CD, URL mappings, DNS

When suggesting changes, explain the trade-offs. Update `docs/ARCHITECTURE.md` if the developer agrees.
