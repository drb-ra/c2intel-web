# C2 Intel Feeds — Web UI

A zero-dependency, single-page web app that reads **all** CSV feeds from
[drb-ra/C2IntelFeeds](https://github.com/drb-ra/C2IntelFeeds) — both verified
and unverified — directly from GitHub raw URLs. No build step, no server, no
dependencies. Deploy to GitHub Pages or Cloudflare Pages in minutes.

---

## Time windows

Per the upstream README, feeds use three windows based on **last observed activity**:

| Suffix | Window | Meaning |
|---|---|---|
| *(none)* | **7 day** | Seen in the last 7 days — most current |
| `-30day` | **30 day** | Seen in the last 30 days |
| `-90day` | **90 day** | Seen in the last 90 days |

---

## Features

### Search modes

- **Search All Feeds** (default) — enter an IP, domain, or keyword and press Enter.
  Fetches all three time windows for every matching feed family in parallel,
  deduplicates by IP + port, and returns **one row per unique indicator**.
- **Single Feed** — browse any individual feed via a grouped dropdown.

### Results & recency

- One collapsed row per unique IOC across all matched feed families
- **Found in** column — each family name is a badge linking to its CSV on GitHub
- **Last seen** column — colour-coded recency pills:
  - `7d` (green) — seen in the last 7 days
  - `30d` (blue) — in 30-day feed but not 7-day (not seen this week)
  - `90d` (purple) — in 90-day feed only (not seen in 30+ days)
  - Dimmed pill = not present in that window
- Results sorted by recency by default (most recent first)

### CobaltStrike beacon config

For any result where the IOC is identified as CobaltStrike and appears in the
30-day feed, a **🛡 CS Config** badge is shown next to the IP. Hovering over
it displays a tooltip with the extracted beacon configuration from
[`C2_configs/cobaltstrike-30day.json`](https://github.com/drb-ra/C2IntelFeeds/blob/master/C2_configs/cobaltstrike-30day.json):

- `BeaconType` — HTTP or HTTPS
- `C2Server` — C2 callback address and URI path
- `Port` — listener port
- `SleepTime` — beacon interval in ms
- `Jitter` — sleep jitter percentage
- `HostHeader` — only shown when non-empty
- `HttpPostUri` — POST callback URI
- `UserAgent` — beacon user-agent string
- `Watermark` — CS licence watermark

The config file is fetched once per session and cached. If an IP has multiple
beacon profiles, all are shown in the tooltip.

### Enrichment links

**IPs** — the IP value links to [Modat](https://magnify.modat.io) (primary data
source). Additional lookup pills in the **Lookup** column:

| Tool | URL format |
|---|---|
| [Modat](https://magnify.modat.io) | `https://magnify.modat.io/hosts/<ip>` |
| [Censys](https://platform.censys.io) | `https://platform.censys.io/hosts/<ip>` |
| [Shodan](https://www.shodan.io) | `https://www.shodan.io/host/<ip>` |
| [IPinfo](https://ipinfo.io) | `https://ipinfo.io/<ip>` |

**Domains** — the domain value links to [Validin](https://app.validin.com).
Additional lookup pills:

| Tool | URL format |
|---|---|
| [Validin](https://app.validin.com) | `https://app.validin.com/detail?type=dom&find=<domain>` |
| [Whois / BigDomainData](https://www.bigdomaindata.com) | `https://www.bigdomaindata.com/search.php?q=<domain>` |

### Search input

- **Auto-defanging** — fanged IOCs are normalised automatically before searching:
  - `185[.]224[.]171[.]28` → `185.224.171.28`
  - `evil[.]domain[.]com` → `evil.domain.com`
  - `hxxps://malware[.]io` → `https://malware.io`
  - `bad[com]` → `bad.com`
  - Leading/trailing whitespace stripped automatically

- **Deep-link URL params** — search state is encoded in the URL so results can
  be linked directly from other tools:
  - `?q=<term>` — pre-populates the search box and runs automatically on load
  - `?cat=<value>` — sets category filter: `all`, `verified`, `unverified`,
    `c2`, `kvm`, `rmm`
  - Example: `?q=185.224.171.28&cat=unverified`

### Other

- Category filter: All / Verified only / Unverified only / C2 / KVM / RMM
- Unverified data warning banner shown automatically when relevant
- Match highlighting in global search results
- Sortable columns (click any header)
- Stats cards: mode, total records, matching IOCs, feed families matched
- Copy-to-clipboard per row (copies IP if present, otherwise domain)
- Light / dark mode — respects system preference, persisted to `localStorage`
- Paginated table: 25 / 50 / 100 / 250 / All rows
- All CSV and JSON files cached in memory — each fetched at most once per session
- No build step, no dependencies, no server required

---

## Deploy to GitHub Pages

1. Push this folder to a **public** GitHub repository
2. Go to repo **Settings** → **Pages** → Source: `main` branch, `/ (root)`
3. Click **Save** — live at `https://<username>.github.io/<repo>/` within seconds

To update: `git add . && git commit -m "update" && git push` — GitHub Pages
redeploys automatically.

> Note: GitHub Pages does not support custom response headers, so the
> `_headers` file (Cloudflare-specific) is ignored but harmless.

---

## Deploy to Cloudflare Pages

### Option A — drag-and-drop (fastest, no account linking required)

1. Log in to [dash.cloudflare.com](https://dash.cloudflare.com)
2. **Workers & Pages** → **Create** → **Pages** → **Upload assets**
3. Drag the `c2intel-web/` folder (or a zip) into the upload area
4. Click **Deploy** — live at `<project>.pages.dev` in seconds

To update: return to the project → **Deployments** → **Upload assets** → drag
the updated folder.

### Option B — connected to GitHub (auto-deploy on push)

1. Push this folder to a GitHub repository
2. Cloudflare Pages → **Create** → **Connect to Git** → select the repo
3. Build settings: leave **Build command** blank, set **Output directory** to `/`
4. **Save and Deploy** — every `git push` triggers a re-deploy automatically

### Option C — Wrangler CLI

```bash
npm install -g wrangler
wrangler login
wrangler pages deploy . --project-name c2intel-feeds
```

---

## File structure

```
c2intel-web/
├── index.html   # entire app — HTML, CSS, and JS in one self-contained file
├── _headers     # Cloudflare Pages security headers (CSP, X-Frame-Options, etc.)
└── README.md
```

---

## Data sources

| Source | URL |
|---|---|
| Verified feeds | `https://raw.githubusercontent.com/drb-ra/C2IntelFeeds/master/feeds/<file>.csv` |
| Unverified feeds | `https://raw.githubusercontent.com/drb-ra/C2IntelFeeds/master/feeds/unverified/<file>.csv` |
| CS beacon configs | `https://raw.githubusercontent.com/drb-ra/C2IntelFeeds/refs/heads/master/C2_configs/cobaltstrike-30day.json` |

No data is stored or proxied. The browser fetches everything directly from
GitHub's CDN. All files are cached in memory for the session.

---

## Unverified data

Unverified feeds contain IOCs that have **not** been confirmed as malicious
infrastructure. They may include legitimate security tooling such as
Interactsh, Hak5 Cloud C2, PiKVM, NanoKVM, and similar. A warning banner is
shown automatically whenever unverified results are displayed.

---

## Notes

- GitHub raw URLs have no CORS restrictions — direct browser fetch works without a proxy.
- The `_headers` file applies `Content-Security-Policy` and hardening headers via Cloudflare Pages.
- Global search fetches up to 4 feed families concurrently to stay within browser connection limits.
- Raw scan data is provided courtesy of [Modat](https://modat.io) from May 2026 onwards.
