# Connections Clone

A browser-based clone of the NYT Connections game, deployed at:
**https://connections.clawdia.stefan.fail**

## What it is

Single-file HTML/CSS/JS app (`index.html`) — no build step, no dependencies. Self-hosted via Caddy (static file_server, public, no auth).

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

The NYT API doesn't allow cross-origin requests from browsers. We route through a same-origin reverse proxy configured in Caddy itself — no third-party proxy service, no CORS issue:

```
handle_path /nyt-api/* {
	rewrite * /svc/connections/v2{uri}
	reverse_proxy https://www.nytimes.com {
		header_up Host www.nytimes.com
		header_up User-Agent "Mozilla/5.0 (compatible; connections-clone-proxy)"
	}
}
```

The app fetches `/nyt-api/YYYY-MM-DD.json` (same origin as the page, so no CORS headers are needed at all).

**Previously used a public CORS-proxy service (`api.codetabs.com`) — dropped 2026-09 after it started failing (522 / missing `Access-Control-Allow-Origin`).** Other proxies tried and failed earlier:
- `corsproxy.io` — free tier blocked to localhost only (403)
- `api.allorigins.win` — NYT blocks their IPs (522)
- `proxy.cors.sh` — rate limited on free tier (429)
- `thingproxy.freeboard.io` — DNS resolution failure

Third-party CORS proxies are inherently fragile (rate limits, outages, IP blocks). Since we control the server this app is hosted on, proxying at the Caddy layer is both more reliable and removes a dependency.

## Tile sizing

Tiles are fixed at **148×72px** on desktop (616px wide grid), fluid equal-width on screens <660px. Font size is set by JS based on character count:
- ≤8 chars → 1rem
- 9–13 → 0.82rem
- 14–18 → 0.68rem
- 19+ → 0.58rem

## Deployment

Self-hosted via Caddy at `/home/openclaw/connections-clone` (static file_server, no build step).

Caddyfile block (`/etc/caddy/Caddyfile`):
```
http://connections.clawdia.stefan.fail:11111 {
	import hsts
	handle_path /nyt-api/* {
		rewrite * /svc/connections/v2{uri}
		reverse_proxy https://www.nytimes.com {
			header_up Host www.nytimes.com
			header_up User-Agent "Mozilla/5.0 (compatible; connections-clone-proxy)"
		}
	}
	root * /home/openclaw/connections-clone
	file_server
}
```

To deploy an update:
```bash
cd /home/openclaw/connections-clone
git pull                       # or edit index.html directly
sudo systemctl reload caddy    # picks up file changes immediately anyway;
                                # reload only needed if Caddyfile itself changed
```

## File structure

```
connections-clone/
├── index.html   # entire app — all HTML, CSS, and JS in one file
└── README.md    # this file
```
