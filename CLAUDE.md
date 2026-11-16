# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up). The entire game lives in one file:

- `index.html` (~1,670 lines): inline `<style>`, markup, and one inline `<script>` IIFE containing all game logic, rendering, audio, and input.
- `README.md`: a one-line placeholder.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Open `index.html` in a browser to run it.

## Running and verifying changes

- Run: open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- There are no automated tests. Verify changes by playing the game in a browser: start a run, check firing/bombs/power-ups, confirm boss fights and stage transitions, and check pause/mute/motion toggles.
- For a quick syntax sanity check of the inline script, extract it and run `node --check` on the extracted file.

## Code layout (inside `index.html`)

The main script is a single `(function () { 'use strict'; ... })();` IIFE. Roughly in order:

1. **DOM references**: `document.getElementById` lookups for HUD, overlay, touch controls, and buttons.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`).
3. **Adaptive quality**: `trackPerf` averages frame time over 45 frames and steps `quality` between 0 (full), 1 (medium), and 2 (low).
4. **Config tables**: `DIFFICULTIES` (easy / normal / hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes that cycle by stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled in order).
5. **Audio**: a Web Audio `sfx` object (`ensureAudio`, `beep`, `noiseBurst`, and named effects). Audio is created lazily on the first user gesture via `ensureAudio()`.
6. **Input**: keyboard (arrows/WASD, fire, bomb, pause), mouse on the canvas, and touch joystick plus fire/bomb buttons.
7. **Game state and simulation**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `endGame`, `togglePause`.
8. **Boss attacks**: `BOSS_PATTERNS` plus helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), driven by `updateBoss` and `killBoss`.
9. **Rendering**: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`. The canvas uses a logical size of `BASE_W` x `BASE_H` (480 x 720), scaled by `resize()`.
10. **Main loop**: `loop(now)` clamps `dt` to 0.05s, calls `trackPerf`, then `update` and `draw` while running and not paused. Started with `requestAnimationFrame`.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. Do not introduce a build step or external dependencies without asking.
- Keep the IIFE and `'use strict'` wrapper. Do not leak new globals; attach new code inside the IIFE.
- Use the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) for tunable values rather than adding magic numbers inline.
- Persist player settings only through the existing `skyfighter.*` `localStorage` keys, and wrap new storage access in the same style as existing code.
- Respect the reduced-motion setting: anything that shakes, flashes, or animates heavily should check `reducedMotion`.
- Accessibility: use `announce()` for screen-reader status text rather than writing to the DOM ad hoc.
- Touch and desktop input are both supported. Any new control needs a keyboard path and a touch path.
- Keep changes minimal and in the existing style (2-space indentation, single quotes in JS, short comment headers like `// ---------- Section ----------`).

## Git workflow

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-vfzum6`). Do not push to other branches without explicit permission.
- Do not create pull requests unless the user explicitly asks.
- Commit only when asked, with clear messages describing the gameplay or code change.
