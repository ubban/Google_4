# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its main deliverable is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in a single HTML file on a `<canvas>`.

Files:

- `index.html` — the entire game: inline CSS (`<style>`), markup, and one inline
  `<script>` wrapped in an IIFE. There are no external dependencies, no bundler,
  and no build step.
- `README.md` — minimal placeholder.
- `CLAUDE.md` — this file.

## Running and checking changes

- There is no package manager, build, test runner, or linter configured. Do not
  invent `npm` scripts or test commands.
- To run the game, open `index.html` in a browser (or serve the directory with any
  static server, e.g. `python3 -m http.server`). Check the change in a real browser
  when it touches rendering, input, or audio.
- For syntax-only verification of the inline script, extract the `<script>` block
  and run `node --check` on it.

## Code structure of `index.html`

The inline script is one IIFE with `'use strict'`. Its sections, in order:

1. **DOM references** — `const xxxEl = document.getElementById(...)` at the top.
2. **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`,
   `STORAGE_REDUCED`, all prefixed `skyfighter.`.
3. **Adaptive performance** — `trackPerf` samples frame time and steps a `quality`
   level (0 full, 1 medium, 2 low).
4. **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss
   HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`
   (grunt, interceptor, tank), and `BOSSES` (six bosses, cycled by stage).
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` map. Audio
   is generated with the Web Audio API; there are no sound files.
6. **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`,
   `showStageBanner`, `screenShake`.
7. **Layout and input** — `resize` (fixed 480×720 logical canvas, scaled to fit),
   the virtual joystick and fire/bomb touch buttons, and keyboard handlers.
8. **Game state** — `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`,
   `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`,
   `endGame`, `togglePause`.
9. **Boss patterns** — `BOSS_PATTERNS` is an array of bullet-pattern functions
   (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`,
   `fireHomingSeeded`, `fireDoubleRing`). `updateBoss` and `killBoss` drive them.
10. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
    Drawing reads the current state; it should not mutate game state.
11. **Main loop** — `loop(now)` clamps `dt` to 0.05 s, calls `update` then `draw`
    while running and unpaused, and reschedules with `requestAnimationFrame`.

## Conventions

- Match the existing style: 2-space indentation, single quotes for JS strings,
  `const` by default, `let` only for mutated state, arrow functions for callbacks,
  and section banners written as `// ---------- Name ----------`.
- Keep everything inside the existing IIFE. Do not add globals; expose nothing on
  `window` unless there is a clear reason.
- Gameplay numbers live in the tuning tables near the top of the script. Prefer
  adding or adjusting values there over scattering magic numbers through update or
  draw code.
- Game state changes belong in `update`-side functions; `draw` and its helpers should
  only read state.
- Preserve accessibility hooks: the `announce()` live region, the reduced-motion
  setting (`body.reduced-motion`, which disables screen shake and flashes), and the
  `prefers-reduced-motion` default.
- Preserve input parity: keyboard (arrows/WASD to move, Space to fire, X/Shift for
  bomb, P/Escape to pause) and touch controls (joystick, fire and bomb buttons) must
  both keep working.
- Persisted values use `localStorage` with the `skyfighter.` prefix. Wrap new
  storage access so the game still runs if storage is unavailable.
- Audio must start from a user gesture. Call `ensureAudio()` from input handlers
  rather than creating an `AudioContext` on load.

## Working in this repo

- Keep changes small and scoped to the request. The whole game is one file, so
  diffs are easy to review; avoid reformatting unrelated sections.
- Do not split `index.html` into modules or add a build step unless asked.
- Do not add dependencies, CDN links, or external assets without an explicit request.
- After a change, confirm the page still loads without console errors and that the
  affected mechanic works in a browser, not just by reading the code.

## Git

- Develop on the branch assigned to the session. Do not push to other branches
  without explicit permission.
- Write commit messages that describe the gameplay or code change in the imperative
  mood, with a short subject line.
- Do not open a pull request unless the user asks for one.
