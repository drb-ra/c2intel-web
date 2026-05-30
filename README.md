# C2 Intel Feeds — Web UI

A zero-dependency, single-page web app that reads **all** CSV feeds from
[drb-ra/C2IntelFeeds](https://github.com/drb-ra/C2IntelFeeds) — both verified
and unverified — directly from GitHub raw URLs.

---

## Time windows

Per the upstream README, the feeds use three time windows based on **last observed activity**:

| Suffix | Window | Meaning |
|---|---|---|
| *(none)* | **7 day** | Seen in the last 7 days — most current |
| `-30day` | **30 day** | Seen in the last 30 days |
| `-90day` | **90 day** | Seen in the last 90 days |

An IOC present in the 7-day feed was active very recently. One only in the
90-day feed has not been seen in over 30 days.

---

## Features

### Search modes
- **Search All Feeds** (default) — enter an IP, domain, or keyword and press Enter;
  fetches **all three time windows** for every matching feed family in parallel,
  deduplicates by IP (+ port when available), and returns one row per unique indicator
- **Single Feed** — browse any individual feed via a grouped dropdown; all three
  time windows available per family

### Results & recency
- One row per unique IOC — collapsed across all feed families that matched
- **Found in** column lists every matching feed family as a badge; each badge
  links to the corresponding CSV file on GitHub
- **Last seen** column shows colour-coded recency pills:
  - `7d` (green) — seen in the last 7 days
  - `30d` (blue) — in the 30-day feed (not in 7d = not seen in past week)
  - `90d` (purple) — in the 90-day feed only (not seen in past 30 days)
  - Dimmed pill = not present in that window
- Results sorted by recency by default (most recent first)

### Enrichment links
Every row includes direct links to external enrichment tools:

**IPs** — the IP value links to [Modat](https://magnify.modat.io) (primary data source),
with additional lookup pills in the **Lookup** column:

| Tool | URL format |
|---|---|
| [Modat](https://magnify.modat.io) | `https://magnify.modat.io/hosts/<ip>` |
| [Censys](https://platform.censys.io) | `https://platform.censys.io/hosts/<ip>` |
| [Shodan](https://www.shodan.io) | `https://www.shodan.io/host/<ip>` |
| [IPinfo](https://ipinfo.io) | `https://ipinfo.io/<ip>` |

**Domains** — the domain value links to [Validin](https://app.validin.com)
with additional lookup pills:

| Tool | URL format |
|---|---|
| [Validin](https://app.validin.com) | `https://app.validin.com/detail?type=dom&find=<domain>` |
| [Whois / BigDomainData](https://www.bigdomaindata.com) | `https://www.bigdomaindata.com/search.php?q=<domain>` |

### Search input
- **Defanging** — fanged IOCs are automatically normalised before searching:
  - `185[.]224[.]171[.]28` → `185.224.171.28`
  - `evil[.]domain[.]com` → `evil.domain.com`
  - `hxxps://malware[.]io` → `https://malware.io`
  - `bad[com]` → `bad.com`
  - Leading/trailing whitespace stripped automatically
- **Deep-link / URL params** — search state is reflected in the URL for linking
  from other tools:
  - `?q=<term>` — pre-populates and auto-runs the search on page load
  - `?cat=<value>` — sets the category filter (`all`, `verified`, `unverified`,
    `c2`, `kvm`, `rmm`)
  - Example: `?q=185.224.171.28&cat=unverified`

### Other
- Category filter: All / Verified only / Unverified only / C2 / KVM / RMM
- Unverified data warning banner shown automatically when unverified results appear
- Match highlighting in global search results
- Sortable columns (click any header)
- Stats cards: mode, total records, matching IOCs, feed families matched
- Copy-to-clipboard per row (copies IP if present, otherwise domain)
- Light / dark mode with system-preference detection, persisted to `localStorage`
- Paginated table (25 / 50 / 100 / 250 / All)
- Response caching — each CSV file is fetched at most once per browser session
- No build step, no dependencies, no server required

---

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

Every `git push` triggers an automatic re-deploy.

### Option C — Wrangler CLI

```bash
npm install -g wrangler
wrangler login
wrangler pages deploy . --project-name c2intel-feeds
```

## Deploy to GitHub Pages

1. Push this folder to a public GitHub repository
2. Go to repo **Settings** → **Pages** → Source: `main` branch, `/ (root)`
3. Save — live at `https://<username>.github.io/<repo>/`

Note: GitHub Pages does not support custom headers, so the `_headers` file
(Cloudflare-specific) is ignored but harmless.

---

## File structure

```
c2intel-web/
├── index.html   # entire app (HTML + CSS + JS, self-contained)
├── _headers     # Cloudflare Pages security headers (ignored by GitHub Pages)
└── README.md
```

---

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
Each file is cached in memory for the duration of the browser session.

---

## Unverified data

Unverified feeds contain IOCs that have **not** been confirmed as malicious C2
infrastructure. They may include legitimate tools such as Interactsh, Hak5
Cloud C2, PiKVM, NanoKVM, etc. A warning banner is shown automatically whenever
unverified data is displayed.

---

## Notes

- GitHub raw URLs do not have CORS restrictions, so direct browser fetch works.
- The `_headers` file adds `Content-Security-Policy` and other security headers
  automatically through Cloudflare Pages.
- Global search fetches feeds in parallel batches of 4 families at a time to
  stay within browser connection limits while keeping search fast.
- Data is provided courtesy of [Modat](https://modat.io) from May 2026 onwards.
