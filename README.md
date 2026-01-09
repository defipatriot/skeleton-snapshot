# SkeletonSwap Pool Snapshot

Automated daily/weekly/monthly/yearly snapshots of SkeletonSwap pool data on Terra.

## Data Structure

```
data/
├── daily/           # Rolling 7 days (overwritten weekly)
│   ├── day-1.csv    # Monday
│   ├── day-2.csv    # Tuesday
│   ├── ...
│   └── day-7.csv    # Sunday
│
├── weekly/          # Aggregated weekly data
│   ├── 2025-W01.csv
│   └── 2025-W02.csv
│
├── monthly/         # Aggregated monthly data
│   ├── 2025-01.csv
│   └── 2025-02.csv
│
└── yearly/          # Aggregated yearly data
    └── 2025.csv
```

## Metrics Captured

**Daily (raw):**
- `tvl_usd` - Total Value Locked
- `volume_24h_usd` - Rolling 24hr volume
- `volume_7d_usd` - Rolling 7-day volume
- `apr_7d` - 7-day APR
- `reserve_0`, `reserve_1` - Token reserves
- `total_share` - Total LP tokens

**Aggregated (weekly/monthly/yearly):**
- `avg_tvl_usd` - Average TVL for period
- `total_volume_usd` - Sum of 24hr volumes
- `avg_apr_7d` - Average APR
- `avg_reserve_0/1` - Average reserves
- `snapshot_count` - Number of data points

---

## Setup Guide (Render)

### Step 1: Create GitHub Repos

You need TWO repos:

1. **Code repo** - Contains this script (e.g., `skeleton-snapshot`)
2. **Data repo** - Where CSV files are stored (e.g., `pool-data`)

#### Create the data repo:
```bash
mkdir pool-data && cd pool-data
git init
mkdir -p data/daily data/weekly data/monthly data/yearly
touch data/.gitkeep
git add .
git commit -m "init"
git branch -M main
# Create repo on GitHub, then:
git remote add origin https://github.com/YOUR_USERNAME/pool-data.git
git push -u origin main
```

#### Create the code repo:
```bash
# Upload these files to a new GitHub repo
```

---

### Step 2: Create GitHub Personal Access Token

1. Go to GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. Click **"Generate new token (classic)"**
3. Give it a name: `render-pool-snapshot`
4. Set expiration (or no expiration)
5. Check the **`repo`** scope (full control of private repositories)
6. Click **Generate token**
7. **COPY THE TOKEN NOW** - you won't see it again!

---

### Step 3: Create Render Cron Jobs

Go to [render.com](https://render.com) and sign in.

#### Create DAILY job:

1. Click **"New"** → **"Cron Job"**
2. Connect your **code repo** (skeleton-snapshot)
3. Configure:
   - **Name:** `pool-snapshot-daily`
   - **Region:** Oregon (or closest)
   - **Branch:** `main`
   - **Runtime:** `Node`
   - **Build Command:** `npm install`
   - **Command:** `node index.js daily`
   - **Schedule:** `59 23 * * *` (23:59 UTC daily)
   - **Instance Type:** Free

4. Click **"Advanced"** → **"Add Environment Variable"**:
   ```
   GITHUB_TOKEN = ghp_xxxxxxxxxxxxxxxxxxxx
   GITHUB_REPO = YOUR_USERNAME/pool-data
   ```

5. Click **"Create Cron Job"**

#### Create WEEKLY job:

1. **New** → **Cron Job**
2. Same settings except:
   - **Name:** `pool-snapshot-weekly`
   - **Command:** `node index.js weekly`
   - **Schedule:** `5 0 * * 1` (00:05 UTC Monday - runs AFTER Sunday's daily)

3. Add same environment variables

#### Create MONTHLY job:

1. **New** → **Cron Job**
2. Same settings except:
   - **Name:** `pool-snapshot-monthly`
   - **Command:** `node index.js monthly`
   - **Schedule:** `10 0 1 * *` (00:10 UTC on 1st of each month)

3. Add same environment variables

#### Create YEARLY job:

1. **New** → **Cron Job**
2. Same settings except:
   - **Name:** `pool-snapshot-yearly`
   - **Command:** `node index.js yearly`
   - **Schedule:** `15 0 1 1 *` (00:15 UTC on January 1st)

3. Add same environment variables

---

### Step 4: Test It

1. In Render, go to your **daily** cron job
2. Click **"Trigger Run"** (top right)
3. Watch the logs
4. Check your **pool-data** repo - you should see new files!

---

## Cron Schedule Reference

| Job | Schedule | Meaning |
|-----|----------|---------|
| Daily | `59 23 * * *` | Every day at 23:59 UTC |
| Weekly | `5 0 * * 1` | Monday at 00:05 UTC |
| Monthly | `10 0 1 * *` | 1st of month at 00:10 UTC |
| Yearly | `15 0 1 1 *` | January 1st at 00:15 UTC |

---

## Local Testing

```bash
# Install
npm install

# Run each mode
node index.js daily
node index.js weekly
node index.js monthly
node index.js yearly

# With GitHub push (set env vars first)
export GITHUB_TOKEN=ghp_xxx
export GITHUB_REPO=yourusername/pool-data
node index.js daily
```

---

## Troubleshooting

**"Failed to parse JSON"**
- API might be down, check: https://dex.warlock.backbonelabs.io/api/pools/phoenix-1

**"Git push failed"**
- Check GITHUB_TOKEN has `repo` scope
- Check GITHUB_REPO format is `username/repo` (no .git)

**No data in weekly/monthly**
- Need at least some daily data first
- Run daily a few times before weekly

**Render job stuck**
- Check logs in Render dashboard
- Free tier has 400 hours/month limit

---

## API Source

Data from Backbone Labs / SkeletonSwap:
```
https://dex.warlock.backbonelabs.io/api/pools/phoenix-1
```
