# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

- The entire game is one file: `index.html` (~1,670 lines). CSS lives in a `<style>` block, and all game logic lives in a single IIFE inside a `<script>` block near the end of the file.
- There is no build step, bundler, package manager, test suite, or linter. Nothing is installed. To run the game, open `index.html` in a browser (or serve the directory with any static server).
- `README.md` is a placeholder and carries no setup instructions.

## Repository layout

```
index.html   # The whole game: markup, CSS, and JS
README.md    # Placeholder
```

## Architecture of `index.html`

The file is organized in this order:

1. **`<head>`**: viewport meta (locks zoom and scaling for mobile), theme color, and an inline SVG favicon.
2. **`<style>`**: dark color scheme. Canvas is letterboxed and centered in `#container`. HUD is a set of pill-shaped `.stat` elements in `#hud`. Also includes a low-health `#vignette`, a `#flash` overlay, and the `heartBeat` animation.
3. **`<body>`**: HUD, pause and mute and motion buttons, the start/pause/game-over `#overlay`, touch controls (`#touchControls` with a joystick `#moveZone` and `#fireBtn` / `#bombBtn`), a boss bar, and an offscreen `#srLive` live region for screen readers.
4. **`<script>`** (single IIFE, `'use strict'`):
   - **DOM refs** are grabbed up front with `getElementById`.
   - **Persistence**: keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Always read and write these through `localStorage` with the existing constants.
   - **Config tables**: `DIFFICULTIES`, `THEMES` (six stage backdrops that cycle), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses), `BOSS_PATTERNS` (attack patterns), and `KILLS_PER_STAGE` (18).
   - **Audio**: synthesized with the Web Audio API (`beep`, `noiseBurst`, `sfx.*`). `ensureAudio()` must run from a user gesture before sound plays.
   - **Input**: keyboard (`keys[e.code]`), touch joystick plus fire and bomb buttons, and mouse (`canvasPointFromEvent`, which lets the mouse steer the ship).
   - **Adaptive quality**: `trackPerf` watches frame time over 45 frames and steps `quality` between 0 (full), 1 (medium), and 2 (low).
   - **Game loop**: `update(dt)` advances state, `draw()` renders, and `loop(now)` runs them through `requestAnimationFrame`.
   - **Gameplay systems**: spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, a Vulcan/Laser system with 3 levels), smart bombs (`useBomb`), the kill-chain score multiplier (`addScore`, `breakChain`), power-ups (`applyPowerUp`: `life`, `bomb`, `weapon`), lives (`loseLife`), and game over (`endGame`).

## Gameplay rules to keep consistent

- **Difficulty** (`DIFFICULTIES`) controls spawn rate, enemy speed, enemy shot chance, starting lives, starting bombs, and boss HP multiplier. Easy, normal, and hard are the three levels.
- **Stages** advance after `KILLS_PER_STAGE` kills, and the stage index selects the theme and boss.
- **Enemies**: grunts appear from stage 1, interceptors weave and are fast, and tanks have 4 HP and appear from stage 2.
- **Score multiplier** builds with consecutive kills and resets on `breakChain()`.
- **Best score** is saved only when a run ends (`endGame`).

## Controls (for reference when changing input code)

| Action | Keyboard | Touch |
|---|---|---|
| Move | Arrow keys or WASD (via `keys`) | Left-side joystick |
| Fire | Hold Space | Fire button (hold) |
| Bomb | `X` or Shift | Bomb button |
| Pause | `P` or Esc | Pause button |
| Steer / fire | Mouse over canvas to steer; hold mouse button to fire | n/a |

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add a build system, framework, or dependency without asking.
- Use `const` and `let`, match the existing 2-space indentation, and keep the single-quote style used in the script.
- Store new persistent settings under the `skyfighter.` prefix and read them with a `localStorage.getItem` fallback, like the existing ones.
- Keep changes compatible with phone-sized viewports and touch input. The canvas is a fixed 480×720 logical size (`BASE_W`, `BASE_H`) scaled by `resize()`, so draw in logical coordinates.
- Respect `reducedMotion`: new screen shake, flashes, or animations should be gated on it.
- Preserve accessibility hooks: announce important state changes through `announce()`, which writes to `#srLive`.

## Verifying changes

There is no automated test or build. Verify by:

1. Opening `index.html` in a browser and playing a run through start, a stage transition, a boss fight, and game over.
2. Checking the browser console for errors. The game should run with no uncaught exceptions.
3. For touch changes, testing in a mobile viewport or device emulation.

Also run a quick syntax sanity check on the inline script, for example by extracting it and running `node --check`, if Node is available.

## Git

- The default working branch for this repo's Claude sessions is `claude/gifted-tesla-xpjkp6`. Commit and push there unless told otherwise.
- Keep commit messages short and descriptive, in the imperative mood, matching the style of the existing history.
