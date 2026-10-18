# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 canvas game, **Sky Fighter**, a Raiden-style vertical shoot-em-up.

- `index.html` is the entire game: markup, CSS (`<style>`, ~lines 10–359), and JavaScript (`<script>`, ~lines 422–1667). There is no build step, no bundler, no package manager, and no test suite.
- `README.md` is a placeholder and has no setup instructions.
- Git history: `4efdad1` is the initial commit; `c4dc8f6` adds the game.

To run it, open `index.html` in a browser, or serve the directory with any static server.

## Architecture

All game logic lives in one IIFE (`(function () { 'use strict'; ... })();`). It has no modules and no globals.

**DOM layout.** `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`), the boss bar, combo and stage banners, the top bar (mute / reduced-motion / pause), touch controls, and `#overlay` (the title/game-over card with difficulty buttons and the start button).

**Config tables (top of the script).** Tune gameplay here rather than inside the update logic:
- `DIFFICULTIES`: spawn rates, enemy speed and fire chance, lives, bombs, and boss HP multiplier for `easy` / `normal` / `hard`.
- `KILLS_PER_STAGE`: kills needed to advance a stage.
- `THEMES`: per-stage sky gradient, cloud and sun colors, and a name.
- `ENEMY_TYPES`: `grunt`, `interceptor`, `tank`, each with size, HP, speed multiplier, score value, and `minStage`.
- `BOSSES`: boss name, palette, and HP multiplier. Bosses cycle by `(stage - 1) % BOSSES.length`.
- `BOSS_PATTERNS`: bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring) used by `updateBoss`.

**Core loop.** `requestAnimationFrame(loop)` calls `update(dt)` and then `draw()`, but only while `running && !paused`. `dt` is clamped to 50 ms. `trackPerf` adjusts the global `quality` level (0 full, 1 medium, 2 low) when frame times drop, and particle counts scale with it.

**State.** Game state is module-level `let` variables, such as `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, and `chain`/`multiplier`. `resetGame()` reinitializes all of them. `endGame()`, `loseLife()`, `killBoss()`, and `togglePause()` handle the state transitions.

**Rendering.** Everything is drawn on a single 2D canvas context, `ctx`. The canvas is scaled to fit the window by `resize()`, relative to a fixed `BASE_W × BASE_H` of 480×720. Drawing helpers include `drawPlane`, `drawBossShip`, and `drawClouds`. Mutable HUD elements are updated through cached `*El` references.

**Audio.** Sound is synthesized with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`). `sfx` maps named effects to these primitives. Audio starts lazily on the first user gesture, so keep that ordering when adding sounds.

**Input.** Keyboard (`keys` map, arrows/WASD, Space to fire, X/Shift for bomb, P/Esc to pause), mouse (`mouseTarget`), and touch (virtual joystick plus FIRE and BOMB buttons). Touch devices are detected with `isTouchDevice`.

**Persistence.** `localStorage` keys are prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, and `reducedMotion`. Every access is a plain `localStorage` call, so guard new reads and writes if they must work in restricted browser contexts.

**Accessibility.** Reduced motion can be set by the user toggle or by `prefers-reduced-motion`, and it gates screen shake, flashes, and pulses. Announcements go through the `#srLive` region via `announce()`.

## Conventions

- Match the existing style: two-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and small named helpers.
- Keep everything in `index.html` unless there is a clear reason to split it. Don't add a build system, framework, or dependency for a change this small.
- Add new tunable numbers to the config tables near the top of the script rather than hard-coding them in update or draw code.
- Preserve the `quality` scaling when adding particle or effect counts.
- Respect the reduced-motion flag for any new screen shake, flash, or pulse effect.
- Use the `announce()` helper for new state changes that a screen reader user should hear.

## Verification

There are no automated tests or linters. Verify changes by:

1. Opening `index.html` in a browser (or serving the directory) and checking the browser console for errors.
2. Playing through the affected path: start, the relevant difficulty, the stage or boss in question, pause/resume, and game over.
3. Checking touch controls with a touch emulator when touch-related code changes.
4. Checking that `localStorage` persistence still works after a reload.

After editing the script block, syntax-check it. A single unbalanced brace breaks the whole game with no build error to point at it:

```sh
sed -n '/^<script>$/,/^<\/script>$/p' index.html | sed '1d;$d' > /tmp/game.js && node --check /tmp/game.js
```

## Git

- Development branch for this work: `claude/gifted-tesla-w6fk8o`.
- Commit messages in this repo use an imperative summary line followed by a body that explains the why.
- Don't open pull requests unless asked.
