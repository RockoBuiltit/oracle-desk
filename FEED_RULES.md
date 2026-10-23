# oracle-feed.json rules (the desk page reads this file)
- Keep the existing keys: generated_at, status, alerts, watchlist, prelaunch_updates, digest, rejections, snapshot_note.
- ALL timestamps are plain ISO 8601 with offset, e.g. 2026-10-23T08:00:00-05:00 (no text suffixes). status.next_scan = next scheduled run (noon/16:00/20:00 scans, 08:00 digest).
- status: state ("idle" | "scanning" | "alert"), last_scan, next_scan, activity[] (short lines about what was checked), alerts_used_today, alert_cap (3), x_reads_left_today.
- digest: date (YYYY-MM-DD), top3 [{name, why}] (MUST be filled by the morning digest), ai[], quantum[] (strings).
- watchlist items: name, ticker, theme ("AI"|"Quantum"), stage, tier (HIGH|WATCH|REJECT), score, score_history [{date, score}], market_cap, fdv, volume_24h, dex_route, next_milestone, milestone_date, last_update, links [{label,url}], upgrade_if, downgrade_if. Keep dex_route short (chain + venue, not a full contract address).
- prelaunch_updates: project, theme, source_type (official|founder|rumor), text, at, url. rejections: name, theme, score, reason, at.
- After writing, run: bash /workspace/oracle/desk/publish.sh  (validates JSON and pushes to GitHub Pages: https://rockobuiltit.github.io/oracle-desk/). Never put secrets or keys in the feed.
