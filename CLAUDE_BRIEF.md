# Build brief: THE ORACLE Visual Desk

## What it is
A single-page, colorful but clean dashboard for THE ORACLE, a crypto intelligence scout that tracks legitimate AI and quantum / post-quantum token launches. It visualizes alerts, the watchlist, pre-launch updates, and digests. It is read-only: no wallets, no trading, no sign-in, no forms that collect secrets.

## Deliverable
- One self-contained file: `oracle-desk.html` (inline CSS + JS, no build step). Vanilla JS or a CDN-loaded lightweight lib is fine.
- It loads `oracle-feed.json` from the same folder (`fetch('./oracle-feed.json')`), re-fetching every 60 seconds. If the fetch fails (e.g. opened via file://), show a "Load feed" button that accepts a dropped/picked JSON file.
- Include a `sample-oracle-feed.json` matching the schema below, clearly labeled EXAMPLE DATA, so the page works on first open.

## Layout (desktop first, responsive down to tablet)
1. **Header bar**: "THE ORACLE" wordmark, last-scan time (Central time), next scan time, HIGH PRIORITY alerts used today (e.g. 1 / 3), and an All / AI / Quantum toggle that filters everything.
2. **Oracle avatar panel (top left)**: an animated Grok Bot character (see Animation).
3. **Alerts strip**: HIGH PRIORITY alert cards, newest first. Each card: name + ticker, theme badge (AI or Quantum), event + date, Oracle Score ring, two or more confirming source links, risks, verified contract (copy button), impostor warning.
4. **Pre-launch radar**: a radial "radar" chart where each pre-launch project is a dot. Distance from center = days to next milestone (unknown dates sit on the outer ring), color = theme, size = score. Click a dot for a side drawer with stage, milestone, score history sparkline, links, and upgrade/downgrade triggers.
5. **Watchlist table**: sortable and filterable (tier, theme, DEX-available-only). Columns: project, ticker, theme, stage, score (with trend arrow), market cap, FDV, 24h volume, DEX route, next milestone, last update.
6. **Pre-launch updates feed**: timeline of updates with source labels (Official / Founder / Rumor in distinct colors).
7. **Digest panel**: latest morning digest, sections for AI and Quantum, and a prominent "TOP 3 TO RESEARCH TODAY".
8. **Rejections log** (collapsible): name + one-line reason.
9. **Footer**: "Research only, not financial advice." plus snapshot timestamp.

## Animation: the moving Grok Bot
- A friendly, minimal robot/orb character drawn in SVG or canvas (original design; do not copy any company's logo).
- States driven by `status.state` in the feed:
  - `scanning`: bot looks side to side, a sweeping radar beam, small floating generic icons (chat bubble, document, chart, magnifier; no brand logos) drifting into it, with a ticker line "Scanning X... checking SafeTrade... reading docs..." cycling through `status.activity` strings.
  - `idle`: slow breathing glow and an occasional blink.
  - `alert`: a quick bounce and pulse in the alert color, and the matching alert card flashes once.
- When the feed's `generated_at` changes, briefly play `scanning` for ~5 s before settling, so it visibly "comes alive" on each update.
- Respect `prefers-reduced-motion` (fall back to static states).

## Visual style
- Dark background (#0b0f1a-ish), soft glassy cards, generous spacing, one clean sans font (Inter or system).
- Theme colors: AI = electric violet (#8b5cf6), Quantum = cyan/teal (#22d3ee). Tiers: HIGH PRIORITY = hot coral (#ff5d73), WATCH = amber (#fbbf24), REJECT = muted gray.
- Smooth micro-animations (card fade-in, number count-ups, score rings filling), never busy. Fun but readable.

## Data schema: oracle-feed.json
```json
{
  "generated_at": "2026-10-22T12:06:00-05:00",
  "status": { "state": "idle|scanning|alert", "last_scan": "ISO", "next_scan": "ISO", "activity": ["Scanning X for AI agent launches", "Checking SafeTrade listings"], "alerts_used_today": 0, "alert_cap": 3, "x_reads_left_today": 980 },
  "alerts": [ { "id": "", "name": "", "ticker": "", "theme": "AI|Quantum", "event": "", "event_date": "ISO|null", "score": 0, "breakdown": { "team": 0, "tech": 0, "backing": 0, "security": 0, "tokenomics": 0, "dev": 0, "social": 0 }, "why_legit": "", "sources": [ { "label": "", "url": "" } ], "risks": [""], "contract": { "chain": "", "address": "", "verified": true }, "impostor_warning": "", "sent_at": "ISO" } ],
  "watchlist": [ { "name": "", "ticker": "", "theme": "AI|Quantum", "stage": "pre-launch|testnet|live|archived", "tier": "HIGH|WATCH|REJECT", "score": 0, "score_history": [ { "date": "YYYY-MM-DD", "score": 0 } ], "market_cap": null, "fdv": null, "volume_24h": null, "dex_route": "", "next_milestone": "", "milestone_date": "ISO|null", "links": [ { "label": "", "url": "" } ], "upgrade_if": "", "downgrade_if": "", "last_update": "ISO" } ],
  "prelaunch_updates": [ { "project": "", "theme": "AI|Quantum", "text": "", "source_type": "official|founder|rumor", "url": "", "at": "ISO" } ],
  "digest": { "date": "YYYY-MM-DD", "ai": [""], "quantum": [""], "top3": [ { "name": "", "why": "" } ] },
  "rejections": [ { "name": "", "theme": "AI|Quantum", "score": 0, "reason": "", "at": "ISO" } ],
  "snapshot_note": "CoinGecko snapshots, times in Central"
}
```
Handle nulls gracefully ("n/a"). Format money as $25.9M / $1.2B. Show times in America/Chicago.

## Rules
- Read-only display. No wallet connect, no trade buttons, no inputs for keys, seeds, or passwords.
- Every source link opens in a new tab with rel="noopener noreferrer".
- Never hardcode real project data; only the clearly labeled sample file.

## Done when
- Opening `oracle-desk.html` served from a folder next to `oracle-feed.json` renders every section, the toggle filters, the table sorts, the radar drawer works, and the bot animates through all three states (add a small hidden debug key, e.g. press "d", to cycle states).
