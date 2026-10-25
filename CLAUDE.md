# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a single-file, browser-based vertical shoot-'em-up (Raiden-style). The entire game — markup, CSS, and JavaScript — lives in `index.html`. There is no build step, bundler, package manifest, test suite, or linter.

Files:

- `index.html` — the whole game (~1,670 lines).
- `README.md` — a one-line title only.

## Running the game

No install or build is needed. Open `index.html` directly in a browser, or serve the directory with any static file server (e.g. `python3 -m http.server`) and visit it.

## File layout of `index.html`

1. `<head>` — meta tags (viewport locked for mobile), an inline SVG favicon, and `<title>Sky Fighter</title>`.
2. `<style>` (around line 10) — all CSS, including the `reduced-motion` body class and touch-control layout.
3. `<body>` markup (around line 362) — `#container` holding the `<canvas id="gameCanvas">`, HUD (`#hud`, `#topBar`, `#bossBarWrap`), transient banners (`#comboPop`, `#stageBanner`), touch controls (`#touchControls`), and the start/pause/game-over `#overlay`.
4. One `<script>` (around line 422) — a single IIFE with `'use strict'`. Everything is private to that closure; nothing is exposed on `window`.

Inside the IIFE, code is grouped by `// ---------- Section ----------` comments. Keep new code in the matching section.

## Architecture of the script

- **Config tables** near the top: `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage, cycled), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six designs, each with its own colors and HP multiplier).
- **Game loop**: a single `requestAnimationFrame` loop (`loop(now)`) clamps `dt` to 50 ms, then calls `update(dt)` and `draw()` only while `running && !paused`. Game logic is frame-rate-scaled by `dt`; keep new timers in seconds or in `dt`-scaled units.
- **Adaptive quality**: `trackPerf` averages the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low). Honor `quality` when adding expensive effects.
- **Bosses**: `BOSS_PATTERNS[bossIndex]` is an array of attack functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, etc.). `updateBoss` runs a telegraph → fire → cooldown cycle. Add new patterns to the matching boss's array rather than branching in `updateBoss`.
- **Weapons and power-ups**: `player.weapon` is `'vulcan'` or `'laser'`, with `player.level` 1–3. `applyPowerUp` handles `life`, `bomb`, and `weapon` pickups; `maybeSpawnPowerUp` controls drop odds.
- **Audio**: a Web Audio `AudioContext` is created lazily in `ensureAudio()` on the first user gesture. Sound effects are in the `sfx` object, built on `beep` and `noiseBurst`. Respect the `muted` flag.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes are not wrapped in try/catch today; if you add new keys, follow the same prefix and keep the default behavior when a key is missing.
- **Accessibility**: `announce()` writes to the `#srLive` live region for screen readers. Use it for important state changes. `reducedMotion` (from the OS preference or the in-game toggle) should gate screen shake and heavy flashes.
- **Input**: keyboard (arrows/WASD move, Space fire, X/Shift bomb, P/Esc pause), mouse (move to steer, click to fire), and touch (left-side joystick zone, FIRE and BOMB buttons). Touch and mouse listeners are attached to the `#touchControls` elements and `canvas`.
- **Responsive sizing**: `resize()` fits the fixed `BASE_W × BASE_H` (480 × 720) logical canvas into the viewport. Work in logical coordinates; do not use CSS pixels for game positions.

## Conventions

- Plain, dependency-free ES2015+ JavaScript. Use `const`/`let`, arrow functions, and template literals as the existing code does. Do not add frameworks or external CDN scripts without a clear reason.
- Keep everything in `index.html` unless the file becomes unmanageable. If you split it, the single-file layout is a deliberate choice worth discussing first.
- Use 2-space indentation, single quotes for strings, and semicolons, matching the existing code.
- Name new config constants in `UPPER_SNAKE_CASE` and new `localStorage` keys with the `skyfighter.` prefix.
- Comments are brief section headers and short explanations of non-obvious logic. Match the existing density.

## Testing and verification

There is no automated test or lint setup. Verify changes by:

1. Opening `index.html` in a browser (desktop and, if touch-related, a mobile viewport or device emulation).
2. Playing through at least one stage transition and one boss fight after gameplay changes.
3. Checking the browser console for errors.
4. Toggling mute, difficulty, and reduced motion, and confirming the change persists across a reload.

If you add a test harness or tooling, document its commands here.

## Git workflow

- Develop on the branch assigned for the session; do not push to `main` directly.
- Write clear, imperative commit messages that describe the gameplay or code change.
- Keep commits focused: one feature, fix, or refactor per commit where practical.
