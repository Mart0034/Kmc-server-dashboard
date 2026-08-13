# KMC Server Dashboard

A single self-contained `index.html` — no build step, no server, no dependencies.
Open it in any browser (works well on a 10" tablet) and it talks directly to your
Pterodactyl panel's **Client API** to show live CPU/memory/disk/network, uptime,
players (parsed from console), alerts (parsed from console), and a console tail.
It also has Start/Restart/Stop/Kill buttons.

## Setup

1. Open `index.html`.
2. Click the gear icon (top right) to open **Settings**.
3. Enter your panel URL (e.g. `https://panel.example.com`) and a **Client API key**
   (Account → API Credentials → Create, in your panel). A read-only key is enough
   unless you also want the power buttons to work.
4. Click **Fetch my servers**, check the ones you want, then **Save & close**.
   Each saved server becomes a tab across the top.

Everything is stored in this browser's `localStorage` only — the key never leaves
your device except to talk to your own panel.

## CORS

Because this is a static page calling your panel's API directly from the browser,
your panel needs to allow that. If "Test connection" fails with a network/CORS-looking
error:

- Serve this file over `http(s)://` on your own network instead of double-clicking it
  (`file://` origins are blocked by more panels than a real origin is). A one-line way:
  `python3 -m http.server 8000` in this folder, then open `http://<your-ip>:8000` on
  the tablet.
- If it's still blocked, add this page's origin to your panel's allowed CORS origins
  (`config/cors.php` or the `ALLOWED_ORIGINS`/`APP_CORS_ALLOWED_ORIGINS` panel setting,
  depending on version) and clear any relevant cache.

## What it shows

- **Status** — online/starting/stopping/offline/suspended, per server, live via websocket.
- **CPU / memory / disk** — current value, meter against your server's limit, and a
  short trend sparkline.
- **Network in/out** — live throughput, computed from the panel's cumulative byte
  counters.
- **CPU & memory over time** — a rolling ~10-15 minute chart (up to 300 samples).
- **Players online** — parsed from `joined the game` / `left the game` console lines
  (Minecraft-style servers). This is a heuristic, not a query to the game itself.
- **Alerts** — console lines matching error/warning/crash patterns, timestamped.
- **Console tail** — the raw live console feed.

## Customizing

It's one HTML file — edit the `<style>` block for look, or the JS at the bottom for
behavior. A few obvious extension points:

- `JOIN_RE` / `LEAVE_RE` / `CRIT_RE` / `ERR_RE` / `WARN_RE` — the regexes that drive
  player tracking and alerts. Tune them for your game/log format.
- `makeTile(...)` calls in `renderServer` — add/remove/reorder the stat tiles.
- `MAX_HISTORY` — how many samples (at ~2s each) are kept for the trend chart.
