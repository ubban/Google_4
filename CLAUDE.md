# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small static web game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. There is no build system, package manager, framework, or test suite.

Files:

- `index.html` — the entire game: markup, CSS (in a `<style>` block), and game logic (in one `<script>` IIFE). About 1,670 lines.
- `README.md` — a one-line title only.

Git history: `Initial commit`, then `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.

## Running it

No build step. Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. Use a modern desktop or mobile browser. There is no install step.

## Architecture of `index.html`

Everything lives in one file, in this order:

1. **`<head>`** — viewport meta (locked, `user-scalable=no`), theme color, and an inline SVG favicon.
2. **`<style>`** — dark color scheme; HUD pills (score, best, multiplier, weapon, hearts, bombs); vignette/flash overlays; overlay menus; touch controls; a `body.reduced-motion` class that disables animations.
3. **Markup** — `<canvas id="gameCanvas">`, HUD elements, boss bar, overlay (title/menu, difficulty buttons, stats), touch controls (`moveZone`/joystick, `fireBtn`, `bombBtn`), and an `aria-live` region (`srLive`) for screen-reader announcements.
4. **`<script>`** — one IIFE with `'use strict'`. Roughly in sections:
   - DOM lookups and storage keys (`skyfighter.*` in `localStorage`: best score, mute, difficulty, reduced motion).
   - Adaptive quality: `trackPerf` samples frame time and steps `quality` (0 full / 1 medium / 2 low) to scale effects.
   - Tuning tables: `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
   - Audio: a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` synthesizers. Sound is muted via the `muted` flag.
   - Input: keyboard (`keys` map; arrows/WASD move, Space fires, X/Shift bomb, P/Esc pause), mouse on the canvas, and touch (joystick + buttons).
   - Layout: `BASE_W = 480`, `BASE_H = 720` logical resolution, scaled to fit the viewport in `resize()`.
   - Game state and loop: `resetGame`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`. `dt` is clamped to 0.05 s. Pausing is handled by `togglePause`.
   - Gameplay: weapons (`fireWeapon`, `applyPowerUp`), smart bombs (`useBomb`), scoring with combo multiplier (`addScore`, `breakChain`), lives (`loseLife`), stage progression, and `endGame`.
   - Bosses: `spawnBoss`, `updateBoss`, `killBoss`, and the `BOSS_PATTERNS` array of bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).

Game state is plain module-level `let` variables inside the IIFE. There are no classes or modules.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a restructure. Do not add a bundler, framework, or dependencies without asking.
- Plain ES2015+ JavaScript, `const`/`let`, arrow functions, and the Canvas 2D API. Match the existing style: two-space indentation, single quotes in JS, `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for top-level constant tables.
- Tuning values (speeds, HP, spawn rates, scores) belong in the existing config tables near the top of the script, not hard-coded in logic.
- Anything that persists must use the `skyfighter.*` `localStorage` keys. Wrap new storage reads/writes if they could fail in private browsing.
- Respect the accessibility hooks already present: `reducedMotion` (also set from `prefers-reduced-motion`), `srLive` announcements for important events, and keyboard controls alongside mouse and touch.
- Keep the layout working on phones (touch controls appear for touch input) and on desktop (canvas scales to the viewport).

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through: start a game, fire, use a bomb, take damage, pick up a power-up, reach a boss, and reach game over. Confirm the best score persists after reload.
3. Check the difficulty buttons, mute, and reduced-motion toggles.
4. On a narrow viewport (or touch emulation), confirm the HUD and touch controls remain usable.

If you can run a headless browser (Playwright/Chromium), a scripted smoke test that loads the page and checks for console errors is a reasonable addition, but do not add test tooling to the repo without asking.

## Git

- Development branch for this work: `claude/gifted-tesla-e06851`. Commit and push there; do not push to other branches without permission.
- Do not open pull requests unless explicitly asked.
- Do not commit secrets or local editor/OS files.
