# Railway Deployment Guide — gym-server

**Purpose:** Deploy the Arcane BOW gym battle server to Railway with persistent SQLite storage.  
**Date:** March 6, 2026  
**Target URL:** https://gym.awizard.dev  
**Repository:** aWizardxch/aWizard-Familiar

---

## 🎯 What is gym-server?

The **gym-server** is an Express.js backend that handles:
- PvE gym battle matchmaking
- AI opponent seed generation  
- Battle outcome verification
- Integration with bow-app tracker API
- aWizard AI chat responses (GitHub Models API)

**Tech Stack:**
- Node.js + TypeScript
- Express.js
- SQLite (battles database)
- GitHub Models API (for AI responses)

---

## 📋 Prerequisites

- GitHub account with access to `aWizardxch/aWizard-Familiar`
- Railway account (https://railway.app) — free tier works
- GitHub Personal Access Token (for GitHub Models API)
- Testnet11 Chia wallet (for gym battle signing)

---

## 🚀 Part 1: Create Railway Account

1. Go to https://railway.app
2. Click **"Sign Up"** or **"Log In"**
3. Choose **"Continue with GitHub"**
4. Authorize Railway to access your repositories

---

## 🎯 Part 2: Create New Project

### 1. Import Repository
1. From Railway dashboard, click **"New Project"**
2. Select **"Deploy from GitHub repo"**
3. Choose **"aWizardxch/aWizard-Familiar"**
4. Click **"Deploy Now"**

### 2. Configure Monorepo Settings
Railway needs to know where gym-server lives in the monorepo:

1. Click on the deployed service
2. Go to **"Settings"** tab
3. Configure:

**Build Settings:**
```yaml
Root Directory: projects/gym-server
Build Command: npm install --legacy-peer-deps && npm run build
Start Command: node dist/index.js
```

**Watch Paths (Optional):**
```
projects/gym-server/**
```
This ensures Railway only redeploys when gym-server files change.

---

## 🔐 Part 3: Environment Variables

### Required Variables:

Go to **"Variables"** tab and add:

```env
# Server Configuration
PORT=3001
NODE_ENV=production

# CORS Configuration (comma-separated allowed origins)
FRONTEND_URL=https://map.awizard.dev,https://bow.awizard.dev

# Battle Tracker Integration
TRACKER_URL=https://bow.awizard.dev/api/tracker

# Database Configuration
DATABASE_PATH=/data/gym.db

# Testnet11 Wallet for Battle Signing
GYM_FINGERPRINT=your_testnet11_wallet_fingerprint_here

# GitHub Models API (for aWizard AI chat)
GITHUB_TOKEN=ghp_your_github_personal_access_token_here

# Optional: Debug logging
DEBUG=gym:*
```

### Getting Required Secrets:

#### 1. GitHub Personal Access Token
1. Go to https://github.com/settings/tokens
2. Click **"Generate new token (classic)"**
3. Name: `Railway gym-server — GitHub Models`
4. Scopes: **No special scopes needed** (leave all unchecked)
5. Expiration: 90 days or "No expiration"
6. Copy token and save to Railway `GITHUB_TOKEN` variable

#### 2. Testnet11 Wallet Fingerprint
```bash
# On your local machine with Chia wallet
chia keys show

# Output:
# Fingerprint: 1234567890
```
Copy the fingerprint to Railway `GYM_FINGERPRINT` variable.

---

## 💾 Part 4: Configure Persistent Volume

gym-server uses SQLite, so we need persistent storage:

### 1. Add Volume
1. Go to **"Settings"** → **"Volumes"**
2. Click **"New Volume"**
3. **Mount Path:** `/data`
4. **Size:** 1 GB (default, can increase later)
5. Click **"Add"**

### 2. Verify Database Path
Ensure `DATABASE_PATH=/data/gym.db` in environment variables matches the mount path.

**Important:** Without a volume, the SQLite database will reset on every deployment!

---

## 🌍 Part 5: Custom Domain Setup

### Option A: Railway Subdomain (Free)
Railway auto-generates a domain like:
```
gym-server-production-abc123.up.railway.app
```

This works but isn't branded.

### Option B: Custom Domain (Recommended)
1. Go to **"Settings"** → **"Domains"**
2. Click **"Custom Domain"**
3. Enter: `gym.awizard.dev`
4. Railway provides CNAME record:
   ```
   CNAME: gym.awizard.dev → your-service.up.railway.app
   ```
5. Add CNAME to your DNS provider
6. Wait 5-10 minutes for SSL certificate provisioning

---

## 🔄 Part 6: Enable Auto-Deploy

### Git Integration:
1. Go to **"Settings"** → **"Service"**
2. **Branch:** `main`
3. **Auto-Deploy:** Enabled (default)
4. **Deploy Trigger:** Push to main branch

### Deployment Flow:
```
git push origin main
  └─> Railway detects change in projects/gym-server/
      └─> Runs npm install + npm run build
          └─> Starts node dist/index.js
              └─> Health check on PORT (3001)
                  └─> Live at gym.awizard.dev ✅
```

---

## 🧪 Part 7: Test Deployment

### 1. Trigger First Deployment
```bash
cd projects/gym-server
git commit --allow-empty -m "test: trigger Railway deployment"
git push origin main
```

### 2. Monitor Build Logs
1. Railway dashboard → **"Deployments"** tab
2. Click on latest deployment
3. Watch build logs stream in real-time

**Expected output:**
```
> gym-server@1.0.0 build
> tsc

Build succeeded
Starting server...
Server listening on port 3001
Connected to database: /data/gym.db
```

### 3. Test Health Endpoint
```bash
curl https://gym.awizard.dev/health

# Expected response:
{
  "status": "ok",
  "uptime": 123.45,
  "database": "connected"
}
```

---

## 📊 Part 8: Monitoring & Logs

### View Runtime Logs:
1. Railway dashboard → **"Logs"** tab
2. Real-time log streaming
3. Filter by level: info, warn, error

### Metrics Dashboard:
1. Click **"Metrics"** tab
2. View:
   - CPU usage
   - Memory usage
   - Network traffic
   - Request rate

### Alerts (Pro Plan):
1. **"Settings"** → **"Observability"**
2. Set up alerts for:
   - High CPU (>80%)
   - High memory (>90%)
   - Service crash (restart count)

---

## 🔧 Part 9: Database Management

### Backup SQLite Database:

#### Option A: Railway CLI
```bash
# Install Railway CLI
npm install -g @railway/cli

# Login
railway login

# Link to project
railway link

# Download database backup
railway run -- cat /data/gym.db > gym-backup-$(date +%Y%m%d).db
```

#### Option B: Manual Export
1. Railway dashboard → **"Data"** tab
2. Click **"Export Volume"**
3. Download `gym.db` file

**Schedule:** Back up weekly or before major deployments.

---

## 🔐 Part 10: Security Checklist

- [ ] `GITHUB_TOKEN` set as environment variable (not hardcoded)
- [ ] `GYM_FINGERPRINT` documented in 1Password
- [ ] CORS restricted to `map.awizard.dev` and `bow.awizard.dev` only
- [ ] HTTPS enforced (automatic via Railway)
- [ ] No sensitive data logged (wallet mnemonics, private keys)
- [ ] Rate limiting enabled on API endpoints
- [ ] Volume backups scheduled

---

## 🎯 Deployment Checklist

- [ ] Railway account created
- [ ] Project imported from GitHub
- [ ] Root directory set to `projects/gym-server`
- [ ] Build/start commands configured
- [ ] Environment variables added (PORT, DATABASE_PATH, GITHUB_TOKEN, etc.)
- [ ] Persistent volume mounted at `/data`
- [ ] Custom domain `gym.awizard.dev` configured
- [ ] SSL certificate active
- [ ] First deployment successful
- [ ] Health endpoint responding
- [ ] CORS settings verified
- [ ] Database backup tested

---

## 🐛 Troubleshooting

### Build Fails: "Cannot find module 'typescript'"
**Solution:** Ensure `npm install --legacy-peer-deps` includes devDependencies during build.

### Server Crashes on Startup
**Solution:** Check logs for missing environment variables. Verify `DATABASE_PATH` points to valid volume mount.

### Database Resets on Redeploy
**Solution:** Verify volume is mounted at `/data`. Check `railway.json` or dashboard settings.

### CORS Errors from Frontend
**Solution:** Verify `FRONTEND_URL` includes exact origin (with protocol):
```env
FRONTEND_URL=https://map.awizard.dev
# NOT: map.awizard.dev (missing https://)
```

### Health Endpoint Returns 502
**Solution:** Railway expects service to listen on `$PORT` environment variable (not hardcoded 3001).

---

## 📚 Reference Links

- **Railway Docs:** https://docs.railway.app
- **Railway CLI:** https://docs.railway.app/develop/cli
- **GitHub Models API:** https://github.com/marketplace/models
- **Express.js Production Best Practices:** https://expressjs.com/en/advanced/best-practice-performance.html

---

## 🔄 Alternative Platforms

If Railway doesn't fit your needs:

### Render.com
- Similar to Railway
- Native SQLite support
- Free tier: 750 hours/month

### Fly.io
- More control (Docker-based)
- Persistent volumes
- Free tier: 3 shared-cpu-1x VMs

### Heroku
- Classic PaaS
- Requires PostgreSQL addon (no SQLite)
- Free tier removed (paid plans only)

---

**Next Steps:**
- Deploy gym-server to Railway
- Test battle API integration with bow-app
- Monitor logs for first 24 hours
- Set up database backup schedule
- Update [DEPLOYMENT_MAP.md](./DEPLOYMENT_MAP.md) with Railway URL
