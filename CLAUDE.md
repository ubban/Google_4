# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines): all markup, CSS (`<style>`), and game logic (`<script>`) live here. There is no build step, no bundler, no package manager, and no external dependencies.
- `README.md`: a stub with only the project title.

There are no tests, linters, or CI configuration in the repo.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. No install step is needed.

## Code structure of `index.html`

- **Markup**: `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), the boss bar (`#bossBarWrap`), combo/stage banners, the top-bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **Styles**: a single `<style>` block. Layout is responsive, and the canvas scales to fit the viewport. `touch-action: none` and `user-select: none` are set on the document.
- **Script**: one IIFE (`'use strict'`) containing all game code. Key areas, roughly in file order:
  - Storage keys (`STORAGE_*`, `skyfighter.*` in `localStorage`) for best score, mute, difficulty, and reduced motion.
  - Config tables: `DIFFICULTIES` (easy/normal/hard), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses), `KILLS_PER_STAGE`, and `BASE_W`/`BASE_H` (logical canvas size, 480×720).
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` mapping. Audio is created lazily on the first user gesture.
  - HUD/feedback helpers: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
  - Input: keyboard (`keydown`/`keyup` on `keys[e.code]`), mouse on the canvas, and touch joystick plus FIRE/BOMB buttons.
  - Game state and simulation: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore`/`breakChain` (combo multiplier), `loseLife`, `endGame`, `togglePause`.
  - Boss attacks: `BOSS_PATTERNS` and its `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring), driven by `updateBoss`; `killBoss` advances the stage.
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
  - Main loop: `loop(now)` via `requestAnimationFrame`, plus the start button handler at the end of the IIFE.

## Conventions

- Keep the game self-contained in `index.html` unless the user asks for a split. Do not add a build system or dependencies without a clear reason.
- Gameplay tuning belongs in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `THEMES`) rather than hard-coded in logic.
- Persist user settings only through the existing `STORAGE_*` keys, and wrap storage access so the game still works if `localStorage` is unavailable.
- Respect the reduced-motion setting when adding screen shake, flashes, or pulsing effects (`reducedMotion`, `screenShake`, `flashHit`).
- Keep the game playable on both desktop (mouse, keyboard) and touch devices. Any new control needs both input paths.
- Keep the `aria-live` announcements (`announce`) in step with important state changes such as stage clears, boss spawns, and game over.

## Controls (for reference when testing)

- Mouse: move to steer, click to fire.
- Keyboard: arrows/WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

## Testing

No automated tests exist. Verify changes by loading `index.html` in a browser and playing through the affected behavior on at least one desktop and one touch-sized viewport. For behavior-sensitive changes, check all three difficulties.
