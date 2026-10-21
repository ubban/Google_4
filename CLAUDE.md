# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game lives in one file, `index.html`: markup, CSS, and a single inline `<script>` wrapped in an IIFE (`'use strict'`). `README.md` is a placeholder and contains no setup instructions.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start, fire, use a bomb, collect power-ups, reach a stage clear, lose all lives, pause, and check the game-over screen.
- Check both desktop (keyboard and mouse) and touch layouts when changing input or HUD code. Touch controls are a virtual joystick on the left, with fire and bomb buttons on the right.

## Code layout (inside `index.html`)

The script is organized by `// ---------- Section ----------` comments. Keep new code in the matching section.

- **DOM references**: `const x = document.getElementById(...)` at the top of the IIFE. Add new elements here.
- **Persistence keys**: `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Reading and writing `localStorage` must keep working when storage is unavailable or throws.
- **Tuning data**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (background palettes per stage), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Prefer changing these tables over hard-coding numbers in logic.
- **Audio**: `ensureAudio()` creates the Web Audio context lazily, after a user gesture. `sfx.*` holds the sound effects, and `beep` and `noiseBurst` are the primitives. Respect the `muted` flag.
- **Input**: keyboard state is held in `keys[e.code]`, mouse position is tracked on the canvas, and touch uses the joystick plus fire and bomb buttons. `firing` is the shared fire state.
- **Game state and logic**: `resetGame()`, `update(dt)`, `applyPowerUp`, `fireWeapon`, `useBomb`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
- **Rendering**: `draw()` and the `draw*` helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Rendering is pure canvas 2D.
- **Main loop**: `loop(now)` clamps `dt` to 0.05 s, calls `trackPerf`, then runs `update` and `draw` only when the game is running and not paused.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Match the existing style: two-space indent, single quotes, `const` by default, and short section comments.
- Time-based logic uses `dt` in seconds. Frame-based counters, such as `telegraph` or `attackTimer`, are used in some boss code, so check the surrounding code before changing one.
- The canvas has a fixed logical size of 480×720 (`BASE_W`, `BASE_H`), scaled by CSS in `resize()`. Coordinates in game logic use this logical size, not the window size.
- Adaptive quality: `quality` (0 full, 1 medium, 2 low) is driven by `trackPerf`. Respect it when adding expensive effects, and keep `reducedMotion` in mind for screen shake and flashes (`screenShake`, `flashHit`, `flashBomb`).
- Accessibility: announce important state changes through `announce()` (the `srLive` live region), and keep ARIA labels on buttons up to date.
- Keep changes minimal and local. Do not refactor unrelated sections while making a targeted change.

## Git

- Develop on the branch specified in the task, and push with `git push -u origin <branch>`.
- Write commit messages that describe the gameplay or code change, not the file list.
- Do not open a pull request unless the user asks for one.
