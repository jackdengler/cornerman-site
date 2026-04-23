# Cornerman

A clean, offline-first mobile feed of MMA fight cards, fighter profiles, and
results. Deployed via GitHub Pages at
<https://jackdengler.github.io/cornerman-site/>.

This repository contains the **built** static site (Vite + React, PWA). Data
is pulled from the sibling
[tapology-reader](https://github.com/jackdengler/tapology-reader) project and
enriched into the JSON files under `data/`.

## What's shipped

```
cornerman-site/
├── index.html               # SPA entry (+ OG/meta, CSP)
├── manifest.webmanifest     # PWA manifest (shortcuts, icons)
├── registerSW.js
├── sw.js                    # Workbox service worker
├── workbox-*.js
├── assets/                  # minified JS/CSS bundles
├── data/                    # JSON consumed by the app
│   ├── meta.json
│   ├── events-upcoming.json
│   ├── events-recent.json
│   ├── events-all.json
│   ├── search-index.json
│   ├── events/{slug}.json
│   ├── fighters/{slug}.json
│   └── forums/{id}.json
├── icons/
├── favicon.svg
└── icon.svg
```

## How caching works

The service worker uses two strategies so the app stays usable offline while
still picking up fresh data quickly:

| File | Strategy | TTL |
|---|---|---|
| `meta.json` | Network-first, 4s timeout | 1 hour |
| `events-*.json`, `events/*`, `fighters/*`, `forums/*` | Stale-while-revalidate | 30 days |
| App shell (JS/CSS/HTML) | Precached by Workbox | until next deploy |

If data looks stale, tap Refresh in the app or hard-reload
(cmd/ctrl+shift+r). The footer shows the `lastUpdated` value from `meta.json`.

## Data contract

`data/meta.json` is the freshness signal:

```json
{ "lastUpdated": "2026-04-23T06:17:00Z", "degraded": false, "buildHash": "abc123" }
```

The app reads `data/events-upcoming.json` on boot, falls back to
`events-all.json` for archive views, and uses `search-index.json` as the
haystack for fuzzy search. See
[tapology-reader](https://github.com/jackdengler/tapology-reader) for the
source-of-truth schema.

## Local preview

```sh
# Any static server rooted at the repo works:
python3 -m http.server 8080
# then visit http://localhost:8080/
```

Because the deployed path is `/cornerman-site/`, asset URLs in `index.html`
are absolute (`/cornerman-site/assets/...`). For a quick local check without
rewriting those, serve from a parent directory and visit
`http://localhost:8080/cornerman-site/`.

## PWA install

The app is installable on iOS Safari (Share → Add to Home Screen) and on
Chrome/Edge/Android. Shortcuts declared in the manifest surface as long-press
actions on the home-screen icon.
