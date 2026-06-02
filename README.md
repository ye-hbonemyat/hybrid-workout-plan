# Hybrid Workout Plan

A single-page **hybrid training tracker** (lifting + running) you can install on your phone like a native app. Tap through each set, track weights for progressive overload, watch tutorials inline, and let the week auto-reset every Monday.

Live: https://ye-hbonemyat.github.io/hybrid-workout-plan/

No backend, no build step, no dependencies — just `index.html`, `manifest.json`, and `sw.js`. All your data stays in `localStorage` on your device.

## The plan

| Day       | Focus                       |
| --------- | --------------------------- |
| Monday    | Upper — Chest Priority      |
| Tuesday   | Easy Run                    |
| Wednesday | Upper — Back / Lat Priority |
| Thursday  | Tempo / Interval Run        |
| Friday    | Lower Body + Core           |
| Saturday  | Upper Hypertrophy — V-Taper |
| Sunday    | Long Run                    |

Each lifting day has a fixed exercise list with prescribed sets, rep ranges, and rest times. Edit the `PLAN` array near the top of the `<script>` block in `index.html` to change exercises, sets, reps, or rest seconds.

## Features

### Workout flow

- **Single-day view**: only one day's exercises render at a time so the screen stays focused.
- **Sticky day-tab strip**: seven pills (Mon–Sun) with each day's `done/total` count, a small dot on today, and a filled accent when a day is fully complete.
- **Tinder-style swipe**: drag the day card horizontally with your finger; release past 80 px (or with a flick) to fly it off-screen and slide the next/previous day in. Edge resistance (rubber-band) at the first and last day. Vertical scrolling is preserved.
- **Tap a tab** or use **ArrowLeft / ArrowRight** on desktop to switch days. The last viewed day is remembered.

### Per-set tracking

- Each set is a row with **`S1`/`S2`/…`label`**, a numeric **weight input** (kg or lb), and a **done toggle**.
- Lifting days lay sets out in a **2-column grid**; run days collapse to a single full-width "Mark complete" button.
- Tap the row to mark a set done. Mark/unmark freely — the week's progress count, day badge, tab badge, and streak all update live.

### Progressive overload

- Weights are stored under separate `weight:` keys that **never get cleared by the weekly reset**. Last week's numbers stay visible as your starting point so you only have to bump them up.
- Type `60` → next week the field still shows `60` and you bump to `62.5`. That's the whole flow.
- Toggle **kg / lb** in the header. The toggle changes the displayed label only; numbers are stored as you typed them, so flipping accidentally never corrupts your data.

### Rest timer

- Completing a lifting set automatically starts a countdown using that exercise's prescribed rest seconds.
- Sticky bottom bar shows time left + which set it's for.
- Color turns amber in the last 10 s. At 0, a Web Audio beep plays and the phone vibrates (`navigator.vibrate`).
- Buttons: **+15s**, **Pause/Resume**, **Stop**.
- Run days have rest = 0 so no timer fires.

### Weekly progress + streak

- Top of the page shows a **progress bar** with `daysDone/7` (a day counts as done only when every set is checked).
- A **streak card** tracks consecutive completed weeks plus your **best** streak. Skipping a week resets the run; finishing seven days in a row counts immediately.
- A **week** key (`YYYY-Wnn`) auto-resets the week's checks on a new calendar week. The streak survives the reset; only your set checks clear.

### Tutorials

- Every exercise has a small **▶ play button** beside its name.
- Tapping it opens an in-app modal. If the exercise has a `videoId` set in `PLAN`, the modal embeds that YouTube video via `youtube-nocookie.com`. If not, the modal shows a "Search on YouTube" button that opens a search for `<exercise> form tutorial` in a new tab.
- All `videoId` fields are blank by default — fill them in as you find tutorials you like and they'll embed inline.
- Modal closes on backdrop tap, the × button, or Esc, and stops playback.

### Theme

- Sun / moon toggle in the header — flips between **light** and **dark**, persisted across reloads.
- Defaults to your system preference (`prefers-color-scheme`) on first load.
- Browser chrome (`theme-color` meta) updates with the theme so Android Chrome's status bar matches.
- An inline script in `<head>` applies the saved theme **before first paint** so there's no flash.

### PWA / offline

- Installable on Android, iOS, and desktop Chrome via the standard "Add to Home Screen" / install flow.
- Service worker (`sw.js`) caches the app shell. Strategy:
  - **Navigations** (HTML): network-first, falls back to cached `index.html` offline.
  - **Same-origin assets**: stale-while-revalidate (instant load from cache, refreshed in the background).
- Bumping `CACHE_NAME` in `sw.js` invalidates already-installed clients on their next visit.

## Project layout

```
.
├── index.html        # All UI + logic
├── manifest.json     # PWA metadata (name, theme, icons)
├── sw.js             # Service worker (offline + installable)
├── icons/
│   ├── icon-192.png
│   ├── icon-512.png
│   └── icon-maskable-512.png
└── README.md
```

## Local development

No tooling required. Serve over HTTP so the manifest and service worker load correctly:

```bash
# Python
python3 -m http.server 8000

# or Node
npx serve .
```

Then open `http://localhost:8000`. To test on your phone over the same Wi-Fi, browse to `http://<your-mac-ip>:8000` from Chrome.

## Deploying on GitHub Pages

This repo is configured to be served from GitHub Pages with no build step.

1. Push to `main`.
2. GitHub → **Settings** → **Pages** → **Build and deployment** → Source: **Deploy from a branch** → Branch: `main` / `/ (root)`.
3. Wait ~30–60 s, then open `https://<user>.github.io/hybrid-workout-plan/`.

All asset paths are **relative** (`./manifest.json`, `./icons/...`, `start_url: "./"`, `scope: "./"`), so the same files work under the `/<repo>/` subpath, on a custom domain, and locally without changes.

After every change you ship that you want installed PWAs to pick up, **bump `CACHE_NAME` in `sw.js`** (e.g. `hybrid-plan-v9` → `v10`). The next visit will swap the new shell in.

## Storage layout

All state lives in `localStorage` on the device. Keys:

| Key                            | Lifetime                  | Notes                                                |
| ------------------------------ | ------------------------- | ---------------------------------------------------- |
| `set:<day>:<exIdx>:<setIdx>`   | Cleared on weekly reset   | `"1"` when the set is done, absent when not.         |
| `weight:<day>:<exIdx>:<setIdx>`| **Persistent**            | The number you typed. Survives both reset paths.     |
| `unit`                         | Persistent                | `"kg"` (default) or `"lb"`.                          |
| `week`                         | Persistent                | `YYYY-Wnn`. Triggers auto-reset on change.           |
| `view`                         | Persistent                | Last-viewed day id (`mon`/`tue`/…) so reload sticks. |
| `theme`                        | Persistent                | `"dark"` or `"light"`.                               |
| `streak`                       | Persistent                | Current consecutive-weeks count.                     |
| `streakBest`                   | Persistent                | All-time best streak.                                |
| `streakWeek`                   | Persistent                | Last week key counted toward streak.                 |
| `open:*`                       | Legacy (cleared on reset) | No longer written; tolerated for old installs.       |

The auto-reset only removes keys that start with `set:` or `open:`. The manual **Reset Week** button does the same and also asks for confirmation. Nothing else is touched.

## Adding tutorial videos

Each exercise in `PLAN` (in `index.html`) accepts an optional `videoId`. Find a YouTube video you like, copy its ID (the bit after `v=` in the URL), and add it:

```js
{ name: "Barbell Bench Press", sets: 4, reps: "5–8", rest: 180, videoId: "vcBig73ojpE" },
```

Save → reload. The play button on that exercise now embeds that specific tutorial via `youtube-nocookie.com/embed/<id>`. Without a `videoId` the play button still works — it just opens YouTube search in a new tab.

## Caveats

- **Per-device storage** — no cloud sync. Switching browsers or devices = fresh start.
- **No history view** — only the latest weight per set is stored. If you want week-over-week graphs, add a date-stamped log alongside the existing `weight:` keys.
- **iOS quirks**: `navigator.vibrate` is a no-op on iOS Safari. PWA install on iOS uses the share-sheet "Add to Home Screen" rather than an install prompt.
- **First audio beep** on iOS / Chrome may be silent until you've tapped something on the page (autoplay policy). Since the timer only starts after you tap a set, in practice the first tap satisfies the gesture requirement.

## License

MIT — do whatever you want with it.
