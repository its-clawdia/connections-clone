# Connections Clone

A browser-based clone of the NYT Connections game, deployed at:
**https://connections-clone-clawdia.surge.sh**

## What it is

Single-file HTML/CSS/JS app (`index.html`) — no build step, no dependencies. Deployed via [Surge.sh](https://surge.sh).

## Features

- **Live puzzle fetching** — loads today's NYT Connections puzzle automatically on open, using the user's local date (not UTC, to handle timezones like PST correctly)
- **Date picker** — 📅 button lets you load any puzzle back to June 12, 2023 (first ever puzzle)
- **LocalStorage cache** — fetched puzzles are cached by date; revisiting is instant. Cached dates appear in the dropdown selector
- **Drag & drop tile rearrangement** — mouse drag and touch both supported; swaps tiles on drop
- **Auto-solve** — reveals all groups one by one with animation; requires confirmation first
- **Dark mode** — auto-enables between 8pm–7am (local time) or if system prefers dark; manual toggle (🌙/☀️) saved to localStorage overrides auto
- **Easter egg** — click the "CONNECTIONS" title 5 times quickly to trigger it
- **One-away hint** — detects when 3 of 4 selected tiles are in the same group
- **Shake animation** on wrong guess

## Data source

Puzzles come from the NYT's internal API:
```
https://www.nytimes.com/svc/connections/v2/YYYY-MM-DD.json
```

This endpoint is not officially public but works without authentication. It returns categories ordered by difficulty (index 0 = yellow/easiest, 3 = purple/hardest). There is no separate `difficulty` field — order is the difficulty.

### CORS proxy

The NYT API doesn't allow cross-origin requests from browsers. We route through:
```
https://api.codetabs.com/v1/proxy?quest=<encoded-url>
```

**Proxies that were tried and failed:**
- `corsproxy.io` — free tier blocked to localhost only (403)
- `api.allorigins.win` — NYT blocks their IPs (522)
- `proxy.cors.sh` — rate limited on free tier (429)
- `thingproxy.freeboard.io` — DNS resolution failure

`api.codetabs.com` is free with no known restrictions as of March 2026. If it breaks, try alternatives or deploy a small Cloudflare Worker proxy.

## Tile sizing

Tiles are fixed at **148×72px** on desktop (616px wide grid), fluid equal-width on screens <660px. Font size is set by JS based on character count:
- ≤8 chars → 1rem
- 9–13 → 0.82rem
- 14–18 → 0.68rem
- 19+ → 0.58rem

## Deployment

Deployed with [Surge.sh](https://surge.sh) under account `flyho@mailbox.org`.

```bash
cd connections
npx surge . connections-clone-clawdia.surge.sh
# enter email + password when prompted
```

## File structure

```
connections/
├── index.html   # entire app — all HTML, CSS, and JS in one file
└── README.md    # this file
```
