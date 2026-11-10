# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project. The only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up, implemented as a single self-contained HTML file.

| Path | Purpose |
| --- | --- |
| `index.html` (~1,670 lines) | The entire game: inline `<style>`, markup, and one `<script>` block (an IIFE in strict mode). No build step, no dependencies. |
| `README.md` | Minimal project readme. |

There is no package manager, test suite, linter, or bundler. Do not add one unless the user asks.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Architecture of `index.html`

Everything lives inside one IIFE. Top to bottom:

1. **DOM references**: `document.getElementById(...)` lookups at the top of the script. Keep new UI elements wired the same way.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Changing a key orphans players' saved settings, so do so deliberately.
3. **Adaptive quality**: `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to keep the frame rate up.
4. **Config tables**: `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six bosses, cycled per stage), `KILLS_PER_STAGE` (18).
5. **Audio**: a Web Audio synth (`beep`, `noiseBurst`, `panFor`). Audio is created lazily through `ensureAudio()`, which must run from a user gesture.
6. **Input**: keyboard (`keys[e.code]`), touch joystick plus fire and bomb buttons, and mouse on the canvas.
7. **Game state and loop**: `resetGame()`, `update(dt)`, and draw functions (`drawPlane`, `drawBossShip`). A single `loop(now)` drives everything through `requestAnimationFrame`.
8. **Boss attacks**: `BOSS_PATTERNS` is an array of per-boss arrays of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).

Coordinates use a fixed logical canvas of **480 x 720** (`BASE_W`, `BASE_H`), scaled to the window by `resize()`.

## Conventions

- **Keep it one file.** The project is intentionally a single HTML file with no build step. Do not split it into modules or add a bundler, framework, or npm dependencies without an explicit request.
- **Match the existing style.** Use `const`/`let`, arrow functions for callbacks, two-space indentation, and single quotes in JavaScript. Keep the strict-mode IIFE wrapper.
- **Gameplay numbers are tuning knobs.** Change them through the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding values in the update logic.
- **Respect accessibility hooks.** Reduced motion (`body.reduced-motion`, `reducedMotion`) and the screen-reader live region (`announce()`) exist for a reason. Route new announcements through `announce()`, and gate new screen-shake or flashing effects on `reducedMotion`.
- **Persist only through the existing keys.** Use `localStorage` with the `skyfighter.*` prefix for any new saved setting.
- **Touch and desktop both matter.** Any new control needs a touch path as well as a keyboard or mouse path.

## Verifying changes

There are no automated tests. To check a change:

1. Load `index.html` in a browser and start a run.
2. Exercise the path you changed: firing, bombs, weapon power-ups, stage transitions, boss fights, and game over (check that the best score persists after a reload).
3. Check the touch layout at phone width, and toggle mute and reduced motion.
4. Watch the browser console for errors.

If you add a headless test harness later, document how to run it here.

## Git

- Develop on the branch the session names (currently `claude/gifted-tesla-98w2dc`). Do not push to other branches without explicit permission.
- Write clear commit messages that say what changed and why.
- Do not open a pull request unless the user asks for one.
