# C2 Intel Feeds — Web UI

A zero-dependency, single-page web app that reads **all** CSV feeds from
[drb-ra/C2IntelFeeds](https://github.com/drb-ra/C2IntelFeeds) — both verified
and unverified — directly from GitHub raw URLs.

## Time windows

Per the upstream README, the feeds use three time windows based on **last observed activity**:

| Suffix | Window | Meaning |
|---|---|---|
| *(none)* | **7 day** | Seen in the last 7 days — most current |
| `-30day` | **30 day** | Seen in the last 30 days |
| `-90day` | **90 day** | Seen in the last 90 days |

An IOC present in the 7-day feed was active very recently. One only in the
90-day feed has not been seen in over 30 days.

## Features

- **Single Feed mode** — browse any of the 60+ feeds (verified + unverified)
  via a grouped dropdown; all three time windows available per family
- **Search All Feeds mode** — enter an IP, domain, or keyword and press Enter;
  the app fetches **all three time windows** for every matching feed family in
  parallel and deduplicates by IOC — one row per unique indicator showing which
  windows it appeared in as colour-coded recency pills:
  - `7d` (green) — seen in the last 7 days
  - `30d` (blue) — in the 30-day feed (not in 7d = not seen in past week)
  - `90d` (purple) — in the 90-day feed only (not seen in past 30 days)
  - Dimmed pill = not present in that window
- Results sorted by recency by default (most recent first)
- Category filter: All / Verified only / Unverified only / C2 / KVM / RMM
- Unverified data warning banner shown automatically
- Live search / filter (single mode) and match highlighting (global mode)
- Sortable columns (click any header)
- Stats cards: mode, total records, matching IOCs, feed families matched
- Copy-to-clipboard per row (copies primary value — IP or domain)
- Light / dark mode with system-preference detection, persisted to localStorage
- Paginated table (25 / 50 / 100 / 250 / All)
- Response caching — each file is fetched at most once per session
- No build step, no dependencies, no server required

## Deploy to Cloudflare Pages (recommended)

### Option A — drag-and-drop (fastest)

1. Log in to [Cloudflare Dashboard](https://dash.cloudflare.com)
2. Go to **Workers & Pages** → **Create** → **Pages** → **Upload assets**
3. Drag the entire `c2intel-web/` folder (or a zip of it) into the upload area
4. Click **Deploy** — your app is live at `<project>.pages.dev` in seconds

### Option B — GitHub-connected deployment

1. Push this folder to a GitHub repository
2. In Cloudflare Pages → **Create** → **Connect to Git**
3. Select your repo
4. Build settings:
   - **Build command**: *(leave blank)*
   - **Build output directory**: `/` (or `.`)
5. Click **Save and Deploy**

Every `git push` will trigger an automatic re-deploy.

### Option C — Wrangler CLI

```bash
npm install -g wrangler
wrangler login
wrangler pages deploy . --project-name c2intel-feeds
```

## File structure

```
c2intel-web/
├── index.html   # entire app (HTML + CSS + JS, self-contained)
├── _headers     # Cloudflare Pages security headers
└── README.md
```

## Data sources

Verified feeds:
```
https://raw.githubusercontent.com/drb-ra/C2IntelFeeds/master/feeds/<file>.csv
```

Unverified feeds:
```
https://raw.githubusercontent.com/drb-ra/C2IntelFeeds/master/feeds/unverified/<file>.csv
```

No data is stored or proxied. The browser fetches directly from GitHub's CDN.
Each file is cached in memory for the duration of the browser session (no
redundant re-fetches when switching between single and global mode).

## Unverified data

Unverified feeds contain IOCs that have **not** been confirmed as malicious C2
infrastructure. They may include legitimate tools such as Interactsh, Hak5
Cloud C2, PiKVM, NanoKVM, etc. A warning banner is shown automatically whenever
unverified data is displayed.

## Notes

- GitHub raw URLs do not have CORS restrictions, so direct browser fetch works.
- The `_headers` file adds `Content-Security-Policy` and other security headers
  automatically through Cloudflare Pages.
- Global search fetches feeds in parallel batches of 6 to stay within browser
  connection limits while keeping search fast.
