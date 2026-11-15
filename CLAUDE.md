# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free browser game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game is a single self-contained file:

- `index.html` (~1,670 lines): inline `<style>`, the HTML/HUD/overlay markup, and one inline `<script>` wrapped in an IIFE with `'use strict'`. No external scripts, stylesheets, fonts, or assets.
- `README.md`: placeholder (title only).

There is no build step, package manager, test suite, linter, or CI configuration.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, exercise the affected feature, and check the console for errors.
- For touch behavior, test with a mobile viewport or touch emulation in dev tools.

## Code layout inside `index.html`

The script is organized into commented sections. Key pieces, roughly in file order:

- **DOM references and storage keys**: `STORAGE_*` constants use the `skyfighter.*` prefix (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
- **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and an `sfx` object built on the Web Audio API. Audio is created lazily on first user gesture.
- **Adaptive performance**: `trackPerf(dt)` (called every frame from the loop).
- **Config tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed and fire chance, lives, starting bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes, one per stage cycle.
  - `ENEMY_TYPES`: `grunt`, `interceptor`, `tank` (with `minStage` gating).
  - `BOSSES`: six bosses (`VANGUARD`, `BEHEMOTH`, `SPECTER`, `DREADNOUGHT`, `WRAITH`, `TITAN`), each with colors and an `hpMul`.
  - `BOSS_PATTERNS`: bullet pattern generators (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **State and lifecycle**: `resetGame()`, `update(dt)`, `draw()`, `loop(now)`, `endGame()`, `togglePause()`, `loseLife()`, `killBoss()`.
- **Gameplay systems**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `applyPowerUp`, `addScore` / `breakChain` (combo multiplier).
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion`, plus HUD helpers (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, `flashBomb`).
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and a touch joystick plus FIRE/BOMB buttons.

Persistence is limited to `localStorage` (best score, mute, difficulty, reduced-motion preference). Access to `localStorage` is not wrapped in try/catch; keep that in mind if you add more storage calls, since it can throw in restricted browser contexts.

## Controls (keep these in sync with the on-screen hint)

- **Desktop**: mouse to steer and click to fire; arrows or WASD to move; Space to fire; X or Shift for a bomb; P or Esc to pause.
- **Mobile**: drag left side to fly; FIRE and BOMB buttons on the right.

If you change a binding, update the `#controlsHint` text in the markup as well.

## Conventions

- Keep the game as a single file unless the user asks for a restructure. Do not add dependencies, frameworks, or a build step without asking.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for config tables and storage keys.
- New gameplay numbers belong in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, etc.) rather than inline magic numbers.
- Respect the reduced-motion path: effects that shake, flash, or pulse should check `reducedMotion` (see `screenShake`, `flashHit`, `flashBomb`).
- Screen-reader announcements go through `announce()` and the `#srLive` region.
- Keep the layout working at phone width. The page is full-screen with `touch-action: none`, so avoid adding elements that need page scrolling.

## Working in this repo

- Default development branch: `claude/gifted-tesla-kokmpl`. Commit and push there unless told otherwise.
- Only create a pull request when the user explicitly asks for one.
- Before committing, load the page in a browser and confirm there are no console errors.
