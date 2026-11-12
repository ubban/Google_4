# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single HTML5 canvas app.

- `index.html` — the entire game: markup, CSS, and one inline `<script>` (wrapped in an IIFE, `'use strict'`). There is no build step, no bundler, no framework, and no external dependencies.
- `README.md` — a stub (title only).

There is no `package.json`, no test suite, and no linter configured. Verification is manual: open the file in a browser.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- Test on desktop (keyboard/mouse) and, when touching input or layout, in a mobile-sized viewport. The UI has touch controls that appear on touch devices.

## Architecture of `index.html`

The script is organized in commented sections, roughly in this order:

1. **DOM refs** — `document.getElementById` lookups for the canvas, HUD, overlay, and touch controls.
2. **Config tables** — `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss designs with colors and HP multipliers).
3. **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read should tolerate a missing or invalid value.
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`) with synthesized `beep` / `noiseBurst` sound effects. Nothing is loaded from disk.
5. **Input** — keyboard (`keys` map keyed by `e.code`), touch joystick plus FIRE/BOMB buttons, and mouse on the canvas.
6. **Game state & logic** — `resetGame`, `update(dt)`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, Vulcan/Laser with 3 levels), bombs (`useBomb`), power-ups (`applyPowerUp`: life, bomb, weapon), kill-chain score multiplier (`addScore`, `breakChain`), and boss attack patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`).
7. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`, explosions). Everything is drawn to a 480×720 logical canvas (`BASE_W` × `BASE_H`) that `resize()` scales to fit the window.
8. **Loop** — `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 50 ms, and calls `update` and `draw` only while running and not paused.

### Game-flow conventions

- Stages are waves of enemies; after `KILLS_PER_STAGE` kills, a boss fight starts. Each stage advances the theme in `THEMES`.
- Enemy and boss tuning lives in the config tables above. Prefer changing those tables over hard-coding numbers in logic.
- The `reducedMotion` setting gates screen shake, hit flash, and pulse effects. New visual effects that shake or flash must respect it.
- Announcements for screen readers go through `announce()` into the `#srLive` region. Keep HUD changes accessible.

## Controls (for reference when testing)

- Move: arrow keys / WASD (via `keys`) or the touch joystick.
- Fire: hold Space or the FIRE button (touch) / mouse button.
- Bomb: `X` or Shift, or the BOMB button.
- Pause: `P` or `Escape`.

## Conventions for editing

- Keep the single-file structure. Do not split out modules, add a build step, or introduce dependencies unless the user asks for it.
- Keep the style consistent with the surrounding code: 2-space indent, single quotes in JS, `const`/`let`, arrow functions for callbacks.
- Put new tuning values in the existing config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than as magic numbers.
- Any new `localStorage` key should use the `skyfighter.` prefix and be wrapped so that failures (private mode, disabled storage) don't break the game.
- Preserve the mobile layout: the `viewport` meta disables zoom, and `touch-action: none` is set on the body and canvas. Don't remove these without a reason.
- Keep the game playable without audio. `ensureAudio()` must be safe to call repeatedly and before any user gesture.

## Git

- Default branch: `main` (on `origin`, `https://github.com/ubban/Google_4`). Work on feature branches and keep commits focused.
- Commit messages: a short imperative summary line, followed by a body explaining the why when it isn't obvious.
- Don't commit generated or machine-specific files; there are currently none in the repo.

## Verification checklist before finishing a change

1. Open `index.html` in a browser with no console errors.
2. Start a game on each difficulty, confirm enemies spawn, firing and bombs work, and power-ups apply.
3. Reach a boss (or temporarily lower `KILLS_PER_STAGE` locally, without committing that change) and confirm the boss bar, patterns, and defeat flow work.
4. Confirm the best score persists across a reload.
5. If you touched layout or input, check a narrow (phone-sized) viewport.
