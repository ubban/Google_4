# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in a single self-contained file, `index.html` (inline `<style>` and one inline `<script>` wrapped in an IIFE). `README.md` is a placeholder and carries no setup instructions.

There is no build step, package manager, test suite, linter, or CI configuration. To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Repository layout

- `index.html`: the whole game.
  - Lines ~8–360: CSS (HUD, overlay card, touch controls, reduced-motion styles).
  - Lines ~361–421: markup (`#container`, `#hud`, `#topBar`, `#touchControls`, `#overlay`).
  - Lines ~422–1668: game script.
- `README.md`: placeholder.

## Architecture of the game script

The script is a single IIFE with `'use strict'`. Everything is a closure-scoped `const`/`let`, so nothing is exposed on `window`. Keep it that way unless there is a clear reason to change it.

Major sections, in file order:

1. **DOM references and storage keys.** `STORAGE_*` constants use the `skyfighter.*` prefix.
2. **Adaptive performance.** `trackPerf` samples frame times and steps `quality` (0 full, 1 medium, 2 low).
3. **Tuning tables.**
   - `DIFFICULTIES`: spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18) controls stage progression.
   - `THEMES`: six sky palettes that cycle by stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (multi-hit, unlocks at stage 2).
   - `BOSSES`: six bosses, cycled in order, each with a `hpMul`.
4. **Audio.** A lazily created `AudioContext` (`ensureAudio`) with synthesized `beep` and `noiseBurst` effects. Nothing is loaded from disk.
5. **HUD and feedback.** Hearts, bombs, combo pop, stage banner, screen shake, screen flash, and an `aria-live` region (`announce`).
6. **Input.** Keyboard (arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause), mouse (move to steer, click to fire), and touch (left-side drag joystick, FIRE and BOMB buttons).
7. **Game state and logic.**
   - `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`.
   - Weapons via `fireWeapon`. Bomb via `useBomb`.
   - Scoring and combos via `addScore` and `breakChain`.
   - Boss attacks are data-driven: `BOSS_PATTERNS` is an array of pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
   - Lifecycle: `loseLife`, `killBoss`, `endGame`, `togglePause`.
8. **Rendering.** `draw()` paints everything onto one 2D canvas each frame. `drawPlane`, `drawBossShip`, and `drawClouds` are the sprite helpers. Shapes are drawn with paths, not image assets.
9. **Main loop.** `loop(now)` caps `dt` at 0.05 s and runs `update` then `draw` only while `running && !paused`.

### Persistence

Four `localStorage` keys store player preferences and the best score:

| Key | Purpose |
| --- | --- |
| `skyfighter.bestScore` | Best score across sessions |
| `skyfighter.muted` | Audio mute flag (`'1'` or `'0'`) |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | Reduced-motion override. If unset, the `prefers-reduced-motion` media query is used. |

All reads and writes are direct and unguarded. If you add storage access, consider wrapping it in try/catch, since some browsers throw when storage is blocked.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a bundler, framework, or package manager without asking.
- **No dependencies.** Use only browser built-ins: Canvas 2D, Web Audio, and DOM APIs.
- **Style.** Match the existing code: 2-space indentation, single quotes in JavaScript, `const` by default, arrow functions for callbacks, and section headers written as `// ---------- Name ----------`.
- **Tuning changes go in the tables.** Prefer adjusting `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` over hard-coding numbers in logic.
- **Coordinates.** Gameplay uses the fixed logical size `BASE_W` × `BASE_H` (480 × 720), scaled by `resize()`. Positions and sizes are in logical units, not CSS pixels.
- **Frame-rate independence.** Movement and timers multiply by `dt` (seconds). Keep new timers in seconds.
- **Accessibility.** Preserve the `aria-live` announcements, the reduced-motion gating for shake and flash, and the touch and keyboard paths when you change input code.

## Verifying changes

There are no automated tests. After changing game code:

1. Open `index.html` in a browser, or serve the directory and open it.
2. Check the browser console for errors on load.
3. Start a game on each difficulty. Confirm movement, firing, bombs, pause (P and Esc), and game over behave correctly.
4. Reach at least one stage transition and a boss fight if the change touches spawning, bosses, or scoring.
5. For touch changes, test with device emulation or a touch-enabled device.

## Git

- The default working branch for this task is `claude/gifted-tesla-oqq40d`.
- Write clear, imperative commit messages that describe the change.
- Do not open pull requests unless the user asks.
