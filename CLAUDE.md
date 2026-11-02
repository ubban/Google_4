# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up), served as a single file.

- `index.html` (~1,670 lines): all markup, CSS, and JavaScript. The game logic lives in one IIFE inside an inline `<script>` near the end of the file.
- `README.md`: a placeholder title only.

There is no build step, package manager, bundler, test suite, linter, or CI config. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`). No dependencies are fetched; everything is inline.

## File layout of `index.html`

| Region | Purpose |
| --- | --- |
| `<head>` meta | Viewport locked for mobile (`user-scalable=no`, `viewport-fit=cover`), `theme-color` `#0b1d3a`. |
| `<style>` (lines ~10–361) | All CSS: HUD, overlay card, touch controls, reduced-motion styles, screen-reader-only helpers. |
| Body markup (lines ~362–421) | `#container` holding `#gameCanvas`, the HUD (`#score`, `#best`, `#mult`, `#weaponName`, `#hearts`, `#bombs`), boss bar, combo/stage banners, top bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (joystick, `#fireBtn`, `#bombBtn`), and the start/pause `#overlay`. |
| `<script>` (lines ~422–1668) | Single `(function () { 'use strict'; ... })()` containing the whole game. |

## Architecture of the game script

Everything is in one closure; there are no modules and no globals to preserve. Major sections, in source order:

1. **DOM refs and storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all under the `skyfighter.*` prefix).
2. **Adaptive performance** (`trackPerf`): samples frame time over 45 frames and steps a `quality` level (0–2) up or down.
3. **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky/cloud palettes, cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`, with `minStage` gating), `BOSSES` (six bosses, cycled by stage, with `hpMul`).
4. **Audio**: lazily created via `ensureAudio()`; sound effects are built from `beep()` and `noiseBurst()` with stereo panning (`panFor`). Mute is persisted.
5. **HUD and feedback**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (ARIA live region).
6. **Layout and input**: `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport. Keyboard (`keydown`/`keyup` into `keys[e.code]`), mouse on the canvas, and touch via a floating joystick plus FIRE/BOMB buttons.
7. **Game state**: `resetGame()`, `update(dt)` (the main simulation step), `applyPowerUp`, `useBomb`, `fireWeapon`, chain/score helpers (`addScore`, `breakChain`), `loseLife`, `endGame`, `togglePause`.
8. **Bosses**: `spawnBoss`, `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, seeded homing, double ring), `updateBoss`, `killBoss`.
9. **Rendering**: `draw()` with helpers `drawPlane`, `drawBossShip`, `drawClouds`. Canvas 2D only.
10. **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw` while running and not paused. Started with `requestAnimationFrame`.

## Conventions to follow

- **Keep it a single file** unless the user asks for a split. Put new CSS in the `<style>` block and new logic inside the existing IIFE.
- **No globals**: keep new code inside the IIFE, and keep `'use strict'`.
- **Use the existing tuning tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) for balance changes rather than hard-coding numbers in the simulation.
- **Persist settings only through the `skyfighter.*` keys**, and wrap any new `localStorage` access so the game still works when storage is unavailable.
- **Respect reduced motion**: effects such as screen shake and flashes should check `reducedMotion` (toggled by `#motionBtn` and the `reduced-motion` body class).
- **Support both input paths**: any new control should work with keyboard, mouse, and touch (the game targets mobile).
- **Accessibility**: announce important state changes through `announce()` rather than only visually.
- **Phone width**: the layout must work at narrow widths without horizontal scroll.

## Verifying changes

There is no automated test setup. To verify a change:

1. Open `index.html` in a browser (or serve the folder) and check the browser console for errors.
2. Play through: start, move, fire, bomb, pause/resume, lose all lives to reach the game-over overlay, and confirm the best score persists after a reload.
3. Reach at least one stage clear and one boss fight when touching combat, boss, or stage logic.
4. Check the mute, reduced-motion, and difficulty toggles still persist across reloads.
5. For layout or CSS changes, check a narrow mobile viewport as well as desktop.

Report what was and was not verified; a visual or gameplay change cannot be confirmed by reading the code alone.

## Git

- Development happens on feature branches; do not push to the default branch without explicit permission.
- Write clear, descriptive commit messages in the imperative mood.
