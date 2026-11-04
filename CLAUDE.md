# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project: a single-page browser game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. It has no build step, no package manager, and no test suite.

Files:

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`, wrapped in one IIFE). About 1,670 lines.
- `README.md` — a one-line title. Nothing else to document there yet.

## Running and verifying

- There is no build, install, or lint step. Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, fire, use a bomb, pause, reach a stage boss, lose all lives, and check the game-over screen.
- Check touch controls as well as keyboard and mouse. The touch overlay is enabled only on touch-capable devices.

## Code structure (`index.html`)

The script is one IIFE with these sections, roughly in file order:

1. **DOM lookups** — every element is fetched once by id at the top of the IIFE. Element ids in the markup must match these lookups.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. These are the only persisted state, all in `localStorage`.
3. **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) up or down.
4. **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
5. **Audio** — a lazily created `AudioContext` (`ensureAudio`) with synthesized `beep` and `noiseBurst` sound effects. Audio starts only after a user gesture.
6. **HUD and effects** — hearts, bombs, combo pop, stage banner, screen shake, flash, and the `announce` helper for screen readers.
7. **Input** — keyboard (`keys` map; P/Escape pause; X/Shift bomb; Space suppressed from scrolling), a virtual joystick and fire/bomb buttons for touch, and mouse move/down on the canvas.
8. **Game state** — `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`.
9. **Boss patterns** — `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, seeded homing, double ring); `updateBoss` and `killBoss` drive them.
10. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Drawing happens on a fixed logical canvas of 480×720 (`BASE_W`, `BASE_H`), scaled by `resize()`.
11. **Main loop** — `loop(now)` runs under `requestAnimationFrame`, computes `dt`, and calls `update` and `draw`.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Adding files means adding a deploy path and possibly a build step, which the project does not have today.
- Keep the code in the existing style: ES2015+ syntax, `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes in JS, and section comments like `// ---------- Name ----------`.
- Put new tuning values in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Preserve accessibility hooks: `announce()` for state changes that screen readers should hear, and the `reducedMotion` setting, which dampens screen shake and turns off the hit/bomb flashes.
- Keep the UI working at phone width. The page uses `touch-action: none`, `user-select: none`, and safe-area insets; do not remove these without a reason.
- Do not add external scripts, fonts, or CDN dependencies without a clear need. The game is currently fully self-contained.

## Persistence

Only `localStorage` is used. Wrap any new storage access in `try`/`catch`, since storage can be blocked (private windows, disabled site data). Use a new `skyfighter.*` key for any new setting.

## Git

- Development branch for this work: `claude/gifted-tesla-5iq1um`. Push with `git push -u origin <branch>`.
- Commit messages in this repo are short, imperative summaries (see `git log`, e.g. "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
