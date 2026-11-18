# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single `<canvas>`.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` IIFE. There is no build step, bundler, or package manager.
- `README.md` — minimal placeholder; it currently contains no project documentation.

## Running the game

No build is required. Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. There are no tests, linters, or CI configured.

## Code structure (`index.html`)

The script is wrapped in a single `(function () { 'use strict'; ... })();` and organized roughly as:

1. **DOM references** — `getElementById` lookups for HUD, overlays, and touch controls.
2. **Persistence keys** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
3. **Config tables** — `DIFFICULTIES` (easy/normal/hard tuning), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`) with `beep` and `noiseBurst` synth helpers.
5. **Input** — keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and touch joystick/fire/bomb buttons.
6. **Game state & logic** — `resetGame`, `update(dt)`, spawning, collisions (`rectsOverlap`), power-ups, bombs, weapons (`fireWeapon`, vulcan/laser), boss attack patterns, `loseLife`, `endGame`, `togglePause`.
7. **Rendering** — `draw()`, `drawPlane`, `drawBossShip`, clouds, explosions, screen shake, flash/vignette effects.
8. **Main loop** — `loop(now)` clamps `dt` to 0.05s, calls `update` and `draw` while running and not paused, and tracks performance via `trackPerf`.

The canvas is sized to a logical `BASE_W × BASE_H` (480×720) and scaled by `resize()`. Keep game logic in logical coordinates; do not depend on CSS pixel size.

## Controls

- Move: arrow keys or WASD
- Fire: Space (hold), or mouse button / touch fire button
- Bomb: X or Shift
- Pause: P or Escape

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Don't introduce a build tool, framework, or dependencies without a clear request.
- **Vanilla JS, ES2015+ syntax**, `const`/`let`, arrow functions, as already used.
- **Tune gameplay through the config tables** (`DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than hard-coding magic numbers in update/draw logic.
- **Respect accessibility settings.** The `reducedMotion` option and the `announce()` live region exist; new visual effects that shake or flash should honor reduced motion.
- **Keep mobile working.** The layout uses `touch-action: none`, safe-area insets, and touch controls. Test any HUD or input change at phone width.
- **Persist only through the existing `skyfighter.*` keys**, and wrap storage access so the game still works if `localStorage` is unavailable.
- Preserve the `'use strict'` IIFE scoping; don't add new globals.

## Verification

Since there is no test suite, verify changes by loading `index.html` in a browser and checking gameplay, HUD updates, pause/resume, game over, and the touch controls where relevant. Check the browser console for errors.

## Git

- Default development branch for automated work: `claude/gifted-tesla-ykbz8q`.
- Write clear, present-tense commit messages describing the change.
