# oracle-feed.json rules (the desk page reads this file)
- Keep the existing keys: generated_at, status, alerts, watchlist, prelaunch_updates, digest, rejections, snapshot_note.
- ALL timestamps are plain ISO 8601 with offset, e.g. 2026-10-23T08:00:00-05:00 (no text suffixes). status.next_scan = next scheduled run (noon/16:00/20:00 scans, 08:00 digest).
- status: state ("idle" | "scanning" | "alert"), last_scan, next_scan, activity[] (short lines about what was checked), alerts_used_today, alert_cap (3), x_reads_left_today.
- digest: date (YYYY-MM-DD), top3 [{name, why}] (MUST be filled by the morning digest), ai[], quantum[] (strings).
- watchlist items: name, ticker, theme ("AI"|"Quantum"), stage, tier (HIGH|WATCH|REJECT), score, score_history [{date, score}], market_cap, fdv, volume_24h, dex_route, next_milestone, milestone_date, last_update, links [{label,url}], upgrade_if, downgrade_if. Keep dex_route short (chain + venue, not a full contract address).
- prelaunch_updates: project, theme, source_type (official|founder|rumor), text, at, url. rejections: name, theme, score, reason, at.
- After writing, run: bash /workspace/oracle/desk/publish.sh  (validates JSON and pushes to GitHub Pages: https://rockobuiltit.github.io/oracle-desk/). Never put secrets or keys in the feed.
- watchlist items also need `price` (current USD price as a number, e.g. 0.000082) for launched tokens; omit or null for pre-launch tokens.

- `deadlines` (top-level list): [{project, theme ("AI"|"Quantum"), title, type (e.g. Airdrop, Snapshot, AMA, TGE, Token sale, Sign-up deadline, Testnet, Mainnet), date (ISO with offset, start), date_end (optional ISO end), all_day (optional true), approx (optional true when the date is estimated), url (official source), status ("open"|"done")}]. Add every sign-up, registration, snapshot, AMA or sale date found. The page marks items DUE SOON within 72 hours and DONE after the end date passes; drop items done for more than 7 days. Alert the user in chat for anything due within 24 hours (outside the 3-alert cap).
- `price` on each watchlist item: numeric USD current price (CoinGecko markets API, snapshot only), null for pre-launch or unlisted tokens. The page no longer shows FDV.

- When the user says "done: <project / title>", set that deadline's `status` to "done" in oracle-feed.json and publish, so every device shows it completed (the page's Mark done button saves on the device it is tapped on, and syncs to other devices through the tick store below).

## Tick store (device sync)
- The page has one setting, `SYNC_URL`, the address of a small store that holds ONLY the deadline "Mark done" ticks, so they match on every device. If it is blank or unreachable the page silently keeps ticks on that device only.
- The store is separate from oracle-feed.json. Scans, digests and summaries NEVER write to it and never change SYNC_URL. Only the page writes to it.
- The "done: <project / title>" chat command still works as before: set that deadline's `status` to "done" in oracle-feed.json and publish.

## Project images
- Every watchlist item carries `logo` ("logos/<slug>.png", 128x128 PNG) and `website` (official site). Get the logo from the project's verified official X profile image or its CoinGecko listing, never an impostor's. publish.sh uploads the logos folder.
- Each digest.top3 entry is {name, why, logo, url} using the same logo path and the official website. The page shows the logo on the card and a Website link in the bottom right; it falls back to the pick number if a logo is missing.

## Morning report
Every day when you produce the 8 AM Central morning report, also copy its contents into a top-level "report" object in oracle-feed.json and publish (the Report button opens a page built from it: full on web, short on phones). Keys: updated (ISO, the report's time), full_report_url (omit if the report only lives in chat), intro, snapshots (snapshot times line), disclaimer, takeaways [strings], prelaunch [{project, ticker, theme, coming, when, score, why}], radar_only [names], tokens [{project, ticker, theme, category, tier, price, market_cap, volume_24h, score, dex}] (every active token, AI and quantum), impostors [{project, theme, fakes, official}], safetrade_source, safetrade [{project, ticker, theme, price, market_cap, volume_24h, score, tier, dex, note}], safetrade_venue {type, x, site, announcements, listing_posts, launchpad, trust_score, steps [{title, text}]}, notes [{section "prelaunch"|"tokens"|"impostors"|"safetrade", theme, text}], sources [{title, url}]. theme is "AI" or "Quantum"; dex is the DEX route name or "" if none; money values are plain USD numbers; leave out FDV.

## SafeTrade pairs and pre-launch list
- Every report.safetrade item carries `pair` (its SafeTrade trading pair from CoinGecko's safe_trade tickers, e.g. "PEARL/USDT"); only list tokens that actually trade on SafeTrade.
- The desk shows only the 10 newest prelaunch_updates; keep the feed to the newest 10 as well and drop older entries when new ones come in.

## Weekly rules (Oct 24 update)
- Near-misses: in the morning report and digest, for any watchlist project close to the HIGH PRIORITY bar (score 75+, credible launch soon or now, 2+ independent high-quality sources), say in one line what it is still missing (undated launch, single unconfirmed post) so the owner knows why no alert fired.
- Own token vs look-alikes: treat a project's own official token separately from wrapped, bridged or look-alike tokens on other chains; mention a look-alike only when it matters, label it unverified unless the official account confirms it, never mix prices or contracts.
- Thin-book spikes: a one-off spike on a thin order book is not a real price; show the 24-hour range and volume and never use a single spike for scores, reports or alerts.
- Price alerts: only when the owner says they hold the token. Record token, amount (if given), price level and venue; check at a reasonable interval; alert once in chat when hit, then remove. Never trade or touch a wallet.
- Never put another chain's look-alike or wrapped token (for example a Solana token with a similar name) on a project's card, top-3 entry, note or DEX route. Use only the project's own facts; mention a look-alike only in the impostors list, or in chat when it matters, labeled unverified.

## Live prices (every 15 minutes)
- Every live (already trading) watchlist item carries `cg_id` (CoinGecko coin id). A routine runs `python3 update_prices.py` every 15 minutes, then publishes. It refreshes price, market_cap, volume_24h, `price_updated` (exact ISO time with seconds) and last_update for those items only, and sets top-level `prices_updated`. The desk shows `price_updated` to the second with a live "x ago" counter.
- When a project starts trading, set its `cg_id` and `price`. Never guess a price; skip tokens with no cg_id.

## Stale milestones
- The bot box and watchlist show next_milestone and milestone_date; the bot box skips milestones whose date has passed. On every scan and digest, move each project's next_milestone and milestone_date forward to its next upcoming dated event and delete events that already happened from the text.
- Never write relative words ("today", "tomorrow", "tonight", "this week") in next_milestone; use weekday and date (e.g. "Season 1 snapshot Sun 10/25"). If an event has a date but no announced time, set milestone_date to 23:59 that day and write "time TBA" in the text. If nothing is upcoming, leave milestone_date empty.

## Open campaigns (Oct 24 update)
-  Open campaigns (Oct 24 update): an open campaign is any points program, airdrop, quest, testnet incentive, waitlist, or snapshot that the owner can join or earn from right now (for example Quip Points, a GenLayer airdrop registration). When you find one, or when an existing one first becomes open, tell the owner in chat the same scan, in plain words, before anything else: what it is, how to join, the deadline or snapshot time if any, the official link (official site or official X account only), and what is unconfirmed. Never leave an open campaign only in the feed or a digest line. Log it in seen.jsonl as an OPEN CAMPAIGN, add it to prelaunch_updates with the label OPEN CAMPAIGN, and repeat it in each morning digest until it closes (include the closing date). Treat referral and shill airdrop links as unsafe and never pass them on; any token claiming the name before an official contract is a scam.

## Built into the desk file (Oct 25 update)
- The shared desk file is fully configured. The setup note in it lists SafeTrade-only items with "pair", the 10-item cap on prelaunch_updates, and the 15-minute live price routine as standing rules (item 13), so update prompts no longer repeat them.
- On an update, owners keep their own SYNC_URL, oracle-feed.json and logos; only the desk html files are replaced.

## Rejections reset (Oct 25 update)
- The `rejections` list holds only the current day. At every 8 AM morning report, empty it first, then add only that report's rejected projects. Scans during the day append to it. Never carry older rejections over.

## Completed deadlines clear every morning (Oct 25 update)
- At each 8 AM morning report, delete from `deadlines` every item whose date (or date_end) has passed, and set `report.updated` to the report time. The page hides completed items (finished by date or ticked by the owner) from before the latest report time, so the "N completed" button disappears until something new completes.

## Alert entry format (Oct 26 fix)
Each item in `alerts` MUST use exactly these types or the desk page stops drawing: name, ticker, theme ("AI"|"Quantum"), event (string), event_date (ISO), score (number), breakdown (OBJECT with numeric keys team, tech, backing, security, tokenomics, dev, social; never a text string), why_legit (string), sources (LIST of {label, url}; never plain strings), risks (LIST of short strings; never one text string), contract (OBJECT {chain, address, verified}; never a text string), impostor_warning (string), market_cap, at, sent_at, status. After writing any alert, check these types before publishing.
