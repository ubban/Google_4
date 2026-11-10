# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based, Raiden-style vertical shoot-'em-up. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: markup (`<body>`), CSS (`<style>`), and JavaScript (one inline `<script>` wrapped in an IIFE with `'use strict'`). About 1,670 lines.
- `README.md` — minimal placeholder (repo title only).

The game renders to a single `<canvas id="gameCanvas">` with the 2D context. Logical resolution is `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the viewport.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests, linters, or build commands. Verify changes by playing the game in a browser. For UI or gameplay changes, exercise the affected path (start, fire, bomb, pause, boss fight, game over) and check the console for errors.
- Preferred headless verification: Playwright with the pre-installed Chromium (see environment notes). Pass `--no-sandbox` if launching Chrome directly.

## Code layout inside `index.html`

Roughly in order of appearance in the script:

1. **DOM refs and storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all under the `skyfighter.*` prefix in `localStorage`).
2. **Adaptive performance** — `trackPerf()` steps a `quality` level (0 full / 1 medium / 2 low) based on a 45-frame rolling average.
3. **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt / interceptor / tank), `BOSSES` (six bosses, cycled per stage).
4. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx.*`, `EXPLODE_SFX`. Audio is created lazily on first user gesture.
5. **HUD and feedback** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce()` (screen-reader live region).
6. **Input** — keyboard (`keys[]`, `keydown`/`keyup`), mouse steering on the canvas, and touch (virtual joystick in `moveZone`, `fireBtn`, `bombBtn`).
7. **Game state** — `resetGame()`, `update(dt)` (main simulation step), `applyPowerUp()`, `useBomb()`, `fireWeapon()`, `addScore()` / combo chain (`breakChain`), `loseLife()`, `killBoss()`, `endGame()`, `togglePause()`.
8. **Boss patterns** — `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring); `updateBoss(dt)`.
9. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`. Drawing is skipped when `running && !paused` is false.
10. **Main loop** — `loop(now)` clamps `dt` to 50 ms, calls `update` then `draw`, and reschedules with `requestAnimationFrame`.

Persistent state lives in `localStorage` only: best score, mute, difficulty, and reduced-motion preference. Reduced motion defaults to the `prefers-reduced-motion` media query until the user toggles it.

## Controls (keep the in-page hint in sync)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Touch: drag left zone to fly, tap FIRE, tap BOMB.

The control hint lives in `#controlsHint` (~line 417). If you change a binding, update that text as well as the `keydown` handler.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split. Do not introduce a build system, bundler, or framework without asking.
- Plain ES2015+ JavaScript; no external dependencies or CDN scripts.
- Preserve the IIFE and `'use strict'`. Use `const`/`let`, arrow functions, and existing helper names.
- Tuning values belong in the constant tables near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`), not scattered as magic numbers.
- Time-based logic uses `dt` in seconds. Keep speeds and timers frame-rate independent.
- Accessibility: respect `reducedMotion` for screen shake and flashes, and route user-facing status changes through `announce()`.
- Mobile: the layout uses `touch-action: none` and safe-area insets; test any HUD change at phone width.

## Git workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-ygj6fe`). Do not push to other branches without permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
