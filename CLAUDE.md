# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), canvas
  element, and all JavaScript (`<script>`, starting ~line 422). No build step, no
  dependencies, no external assets.
- `README.md` — minimal project readme.

There is no package manager, bundler, test suite, linter, or CI configuration in the
repo. Nothing to install or build.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server
(e.g. `python3 -m http.server`) and visit it. No compile step is needed.

## Code layout inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Within it, sections
are marked with `// ---------- Section ----------` comments. Roughly in order:

- **DOM references** — `getElementById` lookups for every HUD, overlay, and touch element.
- **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`
  (all under the `skyfighter.*` prefix in `localStorage`).
- **Adaptive performance** — `trackPerf` steps a `quality` level (0–2) based on rolling
  frame time.
- **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky
  palettes), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, `sfx` (WebAudio synth; no audio files).
- **HUD helpers** — hearts, bombs, combo pops, stage banner, screen shake, flash.
- **Input** — keyboard, touch joystick, fire/bomb buttons, mouse on canvas.
- **Game state & spawning** — `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`,
  `addScore` / `breakChain` (kill-chain multiplier), `useBomb`, `fireWeapon`.
- **Update** — `update(dt)` (per-frame simulation), `applyPowerUp`, `updateBoss`,
  `killBoss`, `loseLife`, `endGame`, `togglePause`.
- **Boss patterns** — `BOSS_PATTERNS` plus the `fire*` helpers (radial, aimed spread,
  columns, spiral, cross, homing, double ring).
- **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
- **Main loop** — `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 50 ms, and only
  calls `update`/`draw` while `running && !paused`.

## Gameplay reference

- **Controls:** move with arrow keys or WASD; fire with Space (hold); bomb with X or Shift;
  pause with P or Esc. On touch devices, a joystick, fire button, and bomb button appear.
- **Weapons:** Vulcan/Laser with 3 power levels, gained via power-ups.
- **Bombs:** limited per run; clear the screen and damage bosses.
- **Stages:** a stage ends after `KILLS_PER_STAGE` kills, which triggers a boss
  (`spawnBoss`, ~1.2 s after the warning). Defeating the boss (`killBoss`) advances
  `stage` and cycles through `THEMES`. Bosses come from `BOSSES` and show a health bar.
- **Difficulty:** easy / normal / hard, set via `DIFFICULTIES` (spawn rate, enemy speed,
  enemy fire chance, starting lives and bombs, boss HP multiplier).
- **Persistence:** best score, mute, difficulty, and reduced-motion preference are saved
  to `localStorage`. Every `localStorage` access should stay wrapped so the game still
  works if storage is unavailable (private mode, blocked site data).

## Conventions

- Keep the game as a **single self-contained HTML file** unless the user asks for a split.
  Do not add npm dependencies or a build step without asking.
- Match the existing style: `const`/`let`, arrow functions, 2-space indentation, single
  quotes in JS, `'use strict'` inside the IIFE.
- Put new tuning numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `THEMES`) rather than scattering magic numbers through the update loop.
- Preserve accessibility hooks: the `srLive` live region (`announce()`), the
  reduced-motion toggle (`body.reduced-motion`), and keyboard operability of the overlay
  buttons.
- Keep the layout working at a 480×720 logical resolution (`BASE_W`, `BASE_H`), scaled
  by `resize()`.

## Verification

No automated tests exist. After changing game logic or rendering, verify by:

1. Opening `index.html` in a browser (or a static server) and checking the console for errors.
2. Playing through start, a stage transition, a boss fight, bomb use, pause/resume, and
   game over.
3. For touch changes, testing with touch emulation in dev tools.

If you add a test harness, document how to run it here.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-gvvx1h`).
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
