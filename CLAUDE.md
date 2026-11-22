# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based Raiden-style vertical shoot-'em-up. The whole game lives in `index.html`: markup, CSS, and one inline `<script>` wrapped in an IIFE. The canvas is a fixed 480×720 logical resolution, scaled to fit the window.

Files:

- `index.html` — the entire game (~1670 lines).
- `README.md` — a one-line placeholder title. Not a source of truth for the game.

There is no build system, package manager, bundler, test suite, or linter. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a modern browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. No build step is required.

## Code layout inside `index.html`

- **`<style>` (about lines 10–359):** dark color scheme, HUD pills, overlay card, touch controls, vignette/flash effects, and the `.reduced-motion` class.
- **`<body>` markup (about lines 360–420):** `#container` holds the canvas, HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), boss bar, combo/stage banners, top buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), a screen-reader live region (`#srLive`), touch controls, and the start/pause/game-over `#overlay`.
- **Script (about lines 422–1667):** one IIFE, with these sections in roughly this order:
  - DOM references and constants: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses with HP multipliers).
  - Persistence and settings: best score, mute, difficulty, and reduced-motion, all in `localStorage`.
  - Audio: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). Audio is created lazily on the first user gesture.
  - Canvas sizing and input: `resize()`, the virtual joystick for touch, mouse/keyboard handling.
  - Game state and loop: `resetGame`, `update(dt)`, `draw()`, `loop(now)`, with `dt` clamped to 0.05 s.
  - Weapons and power-ups: `fireWeapon`, `applyPowerUp` (life / bomb / weapon pickups), `useBomb`.
  - Bosses: `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring) driven by `updateBoss`; `killBoss`.
  - Lifecycle: `loseLife`, `endGame`, `togglePause`.

Keep the single-file structure unless the user asks otherwise. New sections should follow the existing `// ---------- Name ----------` comment style.

## Controls (keep the in-game hint in sync)

- Desktop: mouse moves the ship and click fires; arrow keys or WASD move; Space fires; X or Shift bombs; P or Esc pauses.
- Touch: drag in the left zone to fly; FIRE and BOMB buttons on the right.
- The on-screen hint is in `#controlsHint`. Update it whenever bindings change.

## Persistence keys

All `localStorage` keys use the `skyfighter.` prefix:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every storage access is a plain call with no try/catch. If you add storage access, consider that it may throw in private or restricted contexts.

## Conventions

- Plain HTML, CSS, and vanilla JavaScript (ES2015+ syntax already in use). No frameworks, no dependencies, no CDN scripts.
- Stay inside the existing IIFE and `'use strict'` mode. Don't add globals.
- Game-balance numbers live in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Tune them there rather than inline.
- Respect the reduced-motion setting when adding screen shake, flashes, or pulses. Route them through the existing helpers (`screenShake`, `flashHit`, `flashBomb`) so the toggle applies.
- Keep the UI usable at phone width. The layout already uses `clamp()`, `vw`, and `env(safe-area-inset-*)`; match that.
- Accessibility: announce important state changes through `announce()` into `#srLive`.

## Verification

Since there is no test suite, verify changes by running the game in a browser:

1. Load `index.html` and start a game on each difficulty.
2. Exercise the changed path: firing, bombs, power-ups, boss fight, pause, game over, and best-score persistence after reload.
3. Check the touch layout at a phone-sized viewport.
4. Confirm the browser console shows no errors.

For a quick syntax check of the inline script, you can extract it and run `node --check`.

## Git

- Development branch for this work: `claude/gifted-tesla-5l9qyr`. Commit and push there.
- Don't open a pull request unless explicitly asked.
