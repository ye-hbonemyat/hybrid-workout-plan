# Hybrid Workout Plan

A tiny single-page weekly tracker for a **hybrid training plan** (lifting + running). Tick off each day as you complete it; the week resets automatically every Monday-ish (start of the calendar week). State is persisted in `localStorage`, so it survives reloads on the same device/browser.

Live: https://ye-hbonemyat.github.io/hybrid-workout-plan/

## The plan

| Day       | Focus                  |
| --------- | ---------------------- |
| Monday    | Chest Priority         |
| Tuesday   | Easy Run               |
| Wednesday | Lower + Core           |
| Thursday  | Tempo / Interval Run   |
| Friday    | Back / Lat Priority    |
| Saturday  | Chest + V-Taper        |
| Sunday    | Long Run               |

## Features

- Single static page — pure HTML/CSS/vanilla JS, no build step.
- Weekly progress counter (`x/7`).
- Auto-reset on a new calendar week (year-aware key, so it also resets across the year boundary).
- Manual **Reset Week** button.
- Installable as a PWA (Add to Home Screen) via `manifest.json`.

## Project layout

```
.
├── index.html        # UI + persistence logic
├── manifest.json     # PWA metadata (name, theme, icons)
├── icons/            # PWA icons (add these — see below)
│   ├── icon-192.png
│   ├── icon-512.png
│   └── icon-maskable-512.png
└── README.md
```

## Local development

No tooling required. Just open `index.html`, or serve the folder over HTTP so the manifest loads correctly:

```bash
# Python
python3 -m http.server 8000

# or Node
npx serve .
```

Then visit `http://localhost:8000`.

## Deploying on GitHub Pages

This repo is already configured to be served from GitHub Pages.

1. Push to `main`.
2. GitHub → **Settings** → **Pages** → **Build and deployment** → Source: **Deploy from a branch** → Branch: `main` / `/ (root)`.
3. Wait ~1 minute, then open `https://<user>.github.io/hybrid-workout-plan/`.

All asset paths in `index.html` and `manifest.json` are **relative** (`./manifest.json`, `./icons/...`, `start_url: "./"`, `scope: "./"`), so the same files work under the `/<repo>/` subpath, on a custom domain, and locally.

## Add the PWA icons

The manifest references three icons that aren't in the repo yet. Drop them in `icons/`:

- `icon-192.png` — 192×192
- `icon-512.png` — 512×512
- `icon-maskable-512.png` — 512×512, with safe area for Android adaptive masking

Generators:

- [maskable.app/editor](https://maskable.app/editor) — quick maskable icon from any source PNG.
- `npx pwa-asset-generator <source.png> ./icons` — full set in one shot.

Until icons exist, the install prompt won't appear and DevTools → Application → Manifest will show 404s for them.

## How the persistence works

- Each day's state is saved as a key in `localStorage` (`mon`, `tue`, …, `sun`).
- A `week` key stores the current `${year}-W${weekNumber}` so we can detect a new week.
- On load, `autoReset()` runs first; if the saved week ≠ current week, `localStorage.clear()` wipes last week's checks and writes the new week key.
- Then each checkbox is restored from storage and the progress count is rendered.

## Caveats

- State is per-device/browser — there's no sync.
- The `localStorage.clear()` on auto-reset is fine today (only seven day keys + `week` are stored). If you add other state in the future, switch to removing only the day keys.
- No service worker is registered yet, so the app is **not yet offline-capable** and Chrome/Android may not show an install banner. Adding a tiny `sw.js` is a good next step.

## License

MIT — do whatever you want with it.
