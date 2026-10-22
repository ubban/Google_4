# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up. There is no build step, no package manager, and no test suite. The whole game lives in one file:

- `index.html` (~1670 lines): inline `<style>`, the HTML markup, and one inline `<script>` IIFE containing all game logic, rendering, audio, and input.
- `README.md`: a one-line project title.

Open `index.html` directly in a browser to play. There is nothing to install.

## Repository layout

```
index.html   Entire game: markup (#container, #hud, #overlay, #touchControls), CSS, and JS
README.md    Project title
```

Git: the working branch for this task is `claude/gifted-tesla-xdbtea`. `main` is the default branch.

## Code structure inside `index.html`

The script is wrapped in `(function () { 'use strict'; ... })();`. Keep new code inside that wrapper; do not add globals.

Major sections, in file order (search for the comment or function name):

- **DOM references**: `const` lookups by id at the top of the IIFE (`canvas`, `overlay`, HUD elements such as `score`, `best`, `mult`, `hearts`, `bombs`, plus `startBtn`, `muteBtn`, `motionBtn`, `pauseBtn`).
- **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`. All `localStorage` access is under the `skyfighter.*` prefix.
- **Adaptive performance** (`trackPerf`): averages the last 45 frame times and steps a `quality` level (0 full, 1 medium, 2 low) up or down.
- **Config tables**: `DIFFICULTIES` (spawn rate, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky gradients and stage names, indexed by `(stage - 1) % THEMES.length`), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses with colors and `hpMul`).
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` map. Audio is built on the Web Audio API and is created lazily on the first user gesture.
- **UI helpers**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (writes to the `#srLive` aria-live region).
- **Layout and input**: `resize` (fits a 480x720 logical canvas, `BASE_W` / `BASE_H`), the virtual joystick (`joystickStart/Move/End`), mouse and touch point mapping (`canvasPointFromEvent`), and the keyboard handlers.
- **Game state**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `addScore` (score multiplier and combo chain), `useBomb`, `fireWeapon`, `applyPowerUp`.
- **Boss patterns**: `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss` and `killBoss` drive them.
- **Simulation**: `update(dt)` advances the world each frame. `loseLife`, `endGame`, and `togglePause` handle state transitions.
- **Rendering**: `draw()` plus the sprite helpers `drawPlane`, `drawBossShip`, and `drawClouds`. Rendering is 2D canvas only.
- **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw` while running and not paused.

## Controls (for reference when editing input)

- Keyboard: Arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag the left zone for the virtual joystick, tap FIRE, tap BOMB.

Input is read from `keys[e.code]` (keyed by `KeyboardEvent.code`, not `key`). Keep bindings in sync with the `#controlsHint` text in the overlay.

## Conventions

- Plain HTML, CSS, and vanilla JavaScript. Do not introduce a framework, bundler, or npm dependency unless the user asks for one.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and `// ---------- Section ----------` banner comments to separate sections.
- Tune gameplay through the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in update logic.
- Preserve the accessibility features: the `#srLive` announcements, `prefers-reduced-motion` handling (`reducedMotion` class on `<body>`), and the reduced-motion toggle.
- Persist player settings only through the existing `skyfighter.*` `localStorage` keys.

## Verification

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a desktop browser and play through at least one stage, a boss fight, a pause/resume, and a game over.
2. Check the console for errors. The script runs strict-mode JS and any exception stops the loop.
3. For layout or input changes, test at phone width with touch emulation (joystick, FIRE, BOMB).
4. Check that the best score persists across a reload and that mute and difficulty settings are remembered.

If you cannot run a browser, say so explicitly and verify by reading the code.

## Git workflow

- Make small, focused commits with descriptive messages.
- Push to the branch you were assigned with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
