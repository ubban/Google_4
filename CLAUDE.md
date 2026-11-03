# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>` IIFE). There is no build step, no package manager, no framework, no test suite, and no external assets. Audio is synthesized with the Web Audio API; graphics are drawn on a single `<canvas>` with the 2D context.

Other files:
- `README.md` — a title line plus a stray timestamp line. Leave it alone unless asked to change it.

## How to run it

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and load it from `localhost`.

There are no automated tests and no linter configured. Verify changes by playing the game in a browser (desktop and, when touch-related code changes, a mobile-sized viewport).

## Code layout (inside `index.html`)

Order of the inline script, top to bottom:

1. **DOM references** — `const xxxEl = document.getElementById(...)` for HUD, overlay, touch controls, and buttons.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`).
3. **Helpers** — `announce()` (screen-reader live region), `trackPerf()` (adaptive quality), audio (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object), HUD renderers (`renderHearts`, `renderBombs`), and visual effects (`flashHit`, `screenShake`, `showComboPop`, `showStageBanner`).
4. **Tuning data** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
5. **Input** — keyboard (`keydown`/`keyup` into a `keys` map), mouse/pointer steering and firing on the canvas, and touch (virtual joystick, FIRE, BOMB).
6. **Game state and reset** — `resetGame()` and the `player` object. `score`, `lives`, `bombCount`, and related state are `let` variables in the IIFE scope.
7. **Drawing** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
8. **Spawning and combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `fireWeapon`, `useBomb`, `addScore` / `breakChain` (combo multiplier).
9. **Boss patterns** — `BOSS_PATTERNS` and the `fireRadial` / `fireAimedSpread` / `fireColumns` / `fireSpiralBurst` / `fireCross` / `fireHomingSeeded` / `fireDoubleRing` family, driven by `updateBoss`.
10. **Update** — `update(dt)` advances the simulation each frame; `applyPowerUp` handles pickups.
11. **Game flow** — `loseLife`, `killBoss`, `endGame`, `togglePause`.
12. **Main loop** — `loop(now)` calls `update` then `draw` via `requestAnimationFrame`. `dt` is clamped to 50 ms.

## Game design reference

- **Difficulty** (`DIFFICULTIES`): Easy / Normal / Hard. Each sets spawn rate, enemy speed, enemy fire chance, starting lives (4 / 3 / 3), starting bombs (3 / 2 / 2), and boss HP multiplier. The choice persists in `localStorage`.
- **Stages**: a stage advances every `KILLS_PER_STAGE` (18) kills. Stage N uses `BOSSES[(N-1) % 6]` and `THEMES[(N-1) % 6]`, so both cycle after six stages.
- **Enemies**: `grunt`, `interceptor` (fast, weaving), and `tank` (4 HP, unlocks at stage 2).
- **Weapons**: `vulcan` (spread shot) and `laser` (pierce). Weapon power-ups raise the level from 1 to 3, then switch weapon type.
- **Pickups**: `life`, `bomb`, `weapon`.
- **Score**: base points per kill, multiplied by a combo chain that `breakChain()` resets.
- **Best score** is saved to `localStorage` when a game ends.
- **Adaptive quality**: `trackPerf()` averages the last 45 frame times and steps a `quality` level down when frames run slow, then back up when they recover.
- **Accessibility**: reduced motion (screen shake, flashes, and pulses) is respected via `prefers-reduced-motion` on first load, and can be toggled with the 🎆 button. Live status text goes to the `#srLive` region.

## Controls

| Input | Action |
|---|---|
| Mouse | Move to steer, click to fire |
| Arrows / WASD | Move |
| Space | Fire |
| X or Shift | Bomb |
| P or Esc | Pause |
| Touch | Drag left zone to fly, tap FIRE, tap BOMB |

## Conventions

- **Keep it a single file.** Do not split `index.html` into modules or add a build step unless the user asks. Styles stay in the `<style>` block and logic stays in the single IIFE, so there are no globals to leak.
- **Keep the IIFE intact.** Everything is inside `(function () { 'use strict'; ... })();`. Add new code in the same scope and do not add globals.
- **Tuning lives in data tables.** Prefer adding or changing values in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` over hard-coding numbers inside update or draw logic.
- **Persistence uses the `skyfighter.*` key prefix.** Use `localStorage` only for these small per-player settings, and keep the prefix for any new key.
- **Frame-rate independence.** Movement and timers use `dt` (seconds). Do not assume a fixed 60 FPS.
- **Respect the reduced-motion flag** for any new shake, flash, or pulse effect.
- **Keep the touch and keyboard paths in sync.** New actions should be reachable from both keyboard and touch, and the controls hint in `#controlsHint` should be updated to match.
- **Styling:** dark color scheme (`color-scheme: dark`). HUD colors use the existing gold `#ffd54f` for values and `#7CFC00` for weapon level.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-horgod`) and push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
- Commit messages should be short and descriptive in the imperative mood. The existing history uses lines like "Add Sky Fighter: Raiden-style vertical shoot-em-up game".

## Verification checklist before finishing a change

1. Load `index.html` with no console errors.
2. Start a game on each difficulty, and confirm the HUD updates (score, lives, bombs, weapon).
3. Confirm pause (P / Esc) and resume work.
4. If the change touches boss logic, reach a boss on the stage that uses it and check its patterns and HP bar.
5. If the change touches input, check both mouse/keyboard and touch-sized layouts.
6. Check that the best score and mute/difficulty/motion settings still persist after a page reload.
