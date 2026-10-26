# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up with stages, weapon power-ups, smart bombs, and boss fights. The entire game lives in one file, `index.html` (~1,670 lines), with inline `<style>` and an inline `<script>`. `README.md` is a placeholder.

There is no build step, package manager, bundler, linter, or test suite. Nothing is compiled or installed.

## Running the game

Open `index.html` directly in a browser. No server is required. Use the browser's developer tools for debugging.

## File layout of `index.html`

| Region | Purpose |
| --- | --- |
| `<style>` (lines ~10–360) | All CSS: HUD, overlay, touch controls, boss bar, reduced-motion styles, screen-reader-only class. |
| `<body>` markup (lines ~361–421) | `#container` holding the `<canvas id="gameCanvas">`, HUD (`#hud`), boss bar, stage banner, top bar buttons (mute, reduced motion, pause), touch controls, and the start/pause overlay (`#overlay`). |
| `<script>` (lines ~422–1667) | One IIFE (`'use strict'`) containing all game logic. |

Within the script, the code is organized by section:

- **DOM references**: `const` lookups at the top of the IIFE.
- **Persistence keys**: `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
- **Config tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX`. Audio is synthesized with the Web Audio API, so there are no sound asset files.
- **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
- **Input**: keyboard (`keys` map plus `keydown`/`keyup` listeners), mouse steering and firing via `canvasPointFromEvent`, and touch joystick and buttons.
- **Game state and flow**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `useBomb`, `fireWeapon`, `applyPowerUp`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
- **Boss attacks**: `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
- **Main loop**: `loop(now)` uses `requestAnimationFrame`. `dt` is clamped to 50 ms.

## Conventions

- **Keep it a single file.** New game code goes inside the existing IIFE in `index.html`, unless the user asks to split it. Do not add a build step or dependencies without a request.
- **Match the local style.** Use 2-space indentation, single quotes, `const`/`let`, arrow functions for callbacks, and `function` declarations for named helpers. Config objects are one-line-per-entry tables, aligned as they already are.
- **Tune with the tables.** Difficulty, enemy stats, boss stats, and stage themes are data tables near the top of the script. Prefer changing those over hard-coding numbers in logic.
- **Use the existing helpers.** Reuse `announce()` for screen-reader messages, `screenShake()`/`flashHit()` for feedback, and `beep()`/`noiseBurst()` for sound.
- **Respect reduced motion.** Effects that shake, flash, or pulse must check `reducedMotion`, which defaults to the `prefers-reduced-motion` media query and can be toggled by the user.
- **Persist only through the `STORAGE_*` keys.** Keep the `skyfighter.` prefix.

## Gotchas

- `localStorage` reads and writes are not wrapped in `try/catch`. When storage is blocked (some privacy modes and sandboxed frames), startup may throw at the top of the IIFE. Wrap these accesses if you touch that code.
- `updateMotionBtn`, `updateMuteBtn`, and `setDifficulty` are called during initialization before their `function` declarations appear. This works only because function declarations are hoisted. Do not convert them to `const` arrow functions.
- The frame loop adapts quality through `trackPerf()` and the `quality` variable (0 full, 1 medium, 2 low). Any new per-frame effect should respect that level.
- Touch and mouse input share the same canvas coordinate mapping (`BASE_W` = 480, `BASE_H` = 720, scaled by `resize()`). Keep gameplay coordinates in base units.
- Pause, game-over, and restart flow goes through `togglePause`, `endGame`, and `startBtn`. Changes to state reset should be checked against all three.

## Verifying changes

There are no automated tests. After changing game code, open `index.html` in a browser and check:

1. Start, play, pause, and resume (`P` or `Esc`).
2. Both fire modes: mouse click and Space.
3. A bomb (`X` or `Shift`) and a life loss.
4. Stage transition at 18 kills, and a boss fight.
5. Game over, best-score persistence, and the Start button restarting the run.
6. Mute, reduced motion, and difficulty buttons persist across reloads.
7. The layout on a narrow (phone-width) viewport, including touch controls.

## Git

- Develop on the branch the session specifies. Do not push to other branches without permission.
- Write clear, descriptive commit messages.
