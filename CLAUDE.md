# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, single-page HTML5 game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`. The whole game lives in one file, `index.html` (about 1,670 lines: inline `<style>` in the head, markup in `<body>`, and one inline `<script>` wrapped in an IIFE with `'use strict'`).

Files:

- `index.html` — the entire game (markup, CSS, and JavaScript).
- `README.md` — minimal placeholder; no setup instructions.

There is no `package.json`, bundler, framework, test suite, linter, or CI configuration. Do not add build tooling unless asked.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static file server.
- There are no automated tests or lint commands. Verify changes by loading the page and playing: check the overlay, start/pause, weapons, bombs, bosses, the stage progression, and the mobile touch controls.
- Browser automation is available via the pre-installed Playwright/Chromium if a headless check is useful.

## Code structure (inside `index.html`)

The script is organized by `// ---------- Section ----------` comment banners. Keep new code in the matching section:

- **Setup / DOM refs** — `document.getElementById` lookups at the top of the IIFE.
- **Storage keys** — `skyfighter.*` in `localStorage` (best score, mute, difficulty, reduced motion). Read and write these defensively; they can be missing or blocked.
- **Adaptive performance** — `trackPerf` lowers the `quality` level when frame times are slow.
- **Config tables** — `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`, `BOSS_PATTERNS`. Tune gameplay here rather than hard-coding numbers in logic.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is created lazily after a user gesture.
- **Input** — keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and touch joystick plus FIRE/BOMB buttons.
- **Game state** — `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `applyPowerUp`, `useBomb`, `fireWeapon`, `endGame`, `togglePause`.
- **Boss patterns** — `fireRadial`, `fireAimedSpread`, `fireCross`, etc., selected through `BOSS_PATTERNS`.
- **Rendering** — `draw`, `drawPlane`, `drawBossShip`, `drawClouds`.
- **Loop** — `loop(now)` drives `update` and `draw` with `dt` capped at 0.05 s, via `requestAnimationFrame`.

Conventions observed in the file:

- Plain ES2015+ JavaScript with `const`/`let`, arrow functions, and no modules.
- Game logic uses `dt` in seconds and scales movement by it.
- DOM text updates for screen readers go through `announce()`, which writes to the `srLive` live region.
- Respect `reducedMotion` for screen shake and flash effects.
- Keep the file self-contained; do not split it unless the user asks.

## Controls (for reference when testing)

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire. X or Shift: smart bomb. P or Esc: pause.
- Touch: drag on the left to fly, tap FIRE, tap BOMB.

## Making changes

- Make the smallest change that satisfies the request, and match the existing style, naming, and comment banners.
- Keep the game playable on phone-width screens; the canvas scales through `resize()` to a 480×720 base size.
- After editing, reload the page and exercise the affected feature before reporting it as done.
- Commit and push only when asked, to the branch the session specifies.
