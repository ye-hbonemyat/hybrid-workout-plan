# Hybrid Workout Plan — Codex Guidance

## Project shape

- This is a dependency-free, single-page progressive web app.
- Application UI and JavaScript live in `index.html`.
- `manifest.json` contains PWA metadata and relative asset paths.
- `sw.js` provides the offline cache and must remain same-origin safe.
- Icons live under `icons/`.
- There is no backend, package manager, build step, or test framework.

## Development

- Serve the repository over HTTP; do not test the service worker from `file://`:
  `python3 -m http.server 8000`
- Verify the app at `http://localhost:8000` and use browser DevTools for console, application/storage, manifest, and service-worker checks.
- Keep URLs and PWA paths relative so the app works locally and under the GitHub Pages `/<repository>/` subpath.
- Keep the existing plain HTML/CSS/JavaScript architecture. Do not introduce a framework or dependency without explicit approval.

## Data and behavior invariants

- User data is device-local in `localStorage`; do not add server sync or transmit workout data without explicit product approval.
- `weight:*` keys persist across weekly resets; `set:*` and legacy `open:*` keys may be cleared by reset behavior.
- Preserve the kg/lb display behavior: values are stored exactly as entered and the toggle changes the label only.
- Preserve keyboard navigation, swipe behavior, accessible labels, modal close behavior, theme persistence, and offline behavior.
- Treat embedded tutorial URLs and new-window links as untrusted external content; preserve the existing privacy-oriented YouTube embed host unless explicitly changing it.

## Release and review

- For UI changes, inspect mobile and desktop layouts and verify the affected interaction manually.
- For storage or reset changes, test a fresh install, an existing populated localStorage state, a week transition, manual reset, and reload.
- For PWA or asset changes, bump `CACHE_NAME` in `sw.js` so installed clients receive the new app shell.
- Before handoff, report files changed, manual checks performed, browser limitations, and any migration impact for existing localStorage users.
- Do not commit secrets, generated browser profiles, localStorage exports, or deployment credentials.
