# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html`, with inline CSS (`<style>`, ~lines 10–360), markup (~lines 361–421), and one inline JavaScript block (`<script>`, ~lines 422–1667). `README.md` is a stub.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No install is needed.

## Code layout inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Everything is a closure-scoped `const`/`let`/`function`; nothing is exported to `window`. Keep it that way.

Major sections, in file order (search for the `// ----------` banners and function names):

- **DOM references** – `getElementById` lookups at the top of the IIFE.
- **Persistence keys** – `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). These are the only `localStorage` keys.
- **Adaptive performance** – `trackPerf` samples frame times and steps `quality` (0 full / 1 medium / 2 low).
- **Tuning data** – `DIFFICULTIES` (spawn rates, enemy speed/shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
- **Audio** – a Web Audio synth: `ensureAudio`, `beep`, `noiseBurst`, and the `*SFX` tables. No audio files.
- **HUD and effects** – `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
- **Layout** – `BASE_W = 480`, `BASE_H = 720` logical canvas size; `resize()` scales to the viewport.
- **Input** – keyboard (`keys` map keyed by `e.code`), mouse/pointer steering and fire, and touch joystick plus FIRE/BOMB buttons.
- **Game state** – `resetGame`, `update(dt)` (main simulation), `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`.
- **Boss attacks** – `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring); `updateBoss` and `killBoss`.
- **Rendering** – `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Canvas 2D only.
- **Main loop** – `loop(now)` runs `trackPerf`, then `update` and `draw` when running and not paused. `dt` is clamped to 0.05 s.

## Controls (keep the in-game hint in sync)

The hint text lives in `#controlsHint` (~line 417). If you change a binding, update that string too.

- Keyboard: Arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- Mouse/pointer: move to steer, click to fire.
- Touch: drag left zone to fly, FIRE and BOMB buttons.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, short section banners of the form `// ---------- Name ----------`.
- Add new tuning values to the existing data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in `update`/`draw`.
- Keep rendering to the 2D canvas context. Don't introduce libraries or external CDN scripts without a reason.
- Respect `reducedMotion`: `screenShake` is damped and `flashHit`/`flashBomb` are skipped when it is on. New screen effects should honour it too.
- Accessibility: announce important state changes through `announce()` (the `#srLive` region). Keep buttons labelled.
- Persist new player preferences under the `skyfighter.*` prefix and read them with a fallback default, as the existing `STORAGE_*` reads do.

## Verifying changes

There are no automated tests. To verify a gameplay or rendering change:

1. Open `index.html` in a browser (or serve it locally).
2. Start a run on each difficulty if the change touches tuning.
3. Check the affected path directly: a weapon, boss pattern, power-up, stage transition, pause/resume, game over, and the high-score update.
4. For layout or input changes, check at a phone-sized viewport and with touch emulation.

Say plainly in the final report which of these you did and which you did not.

## Git

- Develop on the branch named in the task. Don't push to `main` directly.
- Commit messages: short imperative subject, optional body explaining the why.
