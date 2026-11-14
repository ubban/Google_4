# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. It is one self-contained file with no build step, no package manager and no test suite.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), canvas element, and one inline `<script>` IIFE (~lines 422–1669) containing all game logic.
- `README.md` — minimal project title only.

There is no dependency installation, bundler, linter or test runner configured. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by loading the page and playing: start a game, check the affected behavior, and check the browser console for errors.
- For mobile-layout changes, check both a desktop viewport and a phone-sized viewport (touch controls only appear when the device reports touch support).

## Code structure inside `index.html`

The script is organized by section comments (`// ---------- Input ----------`, `// ---------- Adaptive performance ----------`, etc.). Key pieces:

- **Config constants**: `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage), `BOSS_PATTERNS` (bullet-pattern functions).
- **Persistence**: `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Keep the prefix and key names stable so saved player data survives.
- **Audio**: Web Audio API, created lazily by `ensureAudio()` (browsers require a user gesture). Sound effects go through `beep()` and `noiseBurst()`.
- **Input**: keyboard (`keys` map keyed by `KeyboardEvent.code`), mouse on the canvas, and touch joystick plus FIRE/BOMB buttons. Controls are documented in the `#controlsHint` element — update it if bindings change.
- **Rendering**: `draw()` runs each frame; `drawPlane()`, `drawBossShip()`, `drawClouds()` handle sprites and parallax. Canvas is a fixed `BASE_W × BASE_H` (480×720) logical size scaled by `resize()`.
- **Game loop**: `loop(now)` calls `update(dt)` then `draw()` via `requestAnimationFrame`. `dt` is clamped to 50 ms. `update()` handles movement, collisions (`rectsOverlap`), spawning, and scoring.
- **Adaptive quality**: `trackPerf()` steps a `quality` level (0–2) up or down based on rolling frame time.
- **State transitions**: `resetGame()`, `loseLife()`, `endGame()`, `killBoss()`, `togglePause()`, `setDifficulty()`.
- **HUD/overlay**: DOM elements are looked up once at the top of the IIFE and updated directly. Screen-reader announcements go through `announce()`.

## Conventions

- Keep the whole game in `index.html` unless the user asks for a split. Do not introduce a framework or external CDN dependency for the game itself.
- Keep the code inside the existing IIFE and `'use strict'` mode. Do not add new globals.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and section comment banners.
- When adding a game object type, follow the pattern of `ENEMY_TYPES` / `BOSSES`: declare data in a config table and keep per-type behavior in small functions.
- Any new persistent setting should get a `skyfighter.*` localStorage key and a `try`-safe read with a sensible default.
- Respect `reducedMotion`: new screen-shake, flash, or heavy particle effects should be gated by it.
- Keep touch and keyboard controls in parity when adding new actions.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-jy32ne`). Do not push to `main` without explicit permission.
- Commit messages in this repo use a short imperative summary line, optionally followed by a bulleted body describing the change.
- Do not create pull requests unless the user explicitly asks for one.
