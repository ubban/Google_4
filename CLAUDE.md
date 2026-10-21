# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file, `index.html` (about 1,670 lines): inline CSS in `<style>` and one IIFE-wrapped inline `<script>` that drives a 2D `<canvas>`. There is no build step, no bundler, no package manager, and no test suite.

Files:
- `index.html`: the entire game (markup, styles, and logic).
- `README.md`: a one-line project title.
- `CLAUDE.md`: this file.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests, linters, or build commands configured. Verify changes by playing the game in a browser, covering at least: starting a run, firing, bombs, a stage transition, a boss fight, game over, and the high-score update.
- Check mobile behavior too (touch joystick, FIRE and BOMB buttons). The layout scales to the viewport.

## Code layout inside `index.html`

The script is one IIFE. Roughly in order:

1. **DOM references**: `const` lookups by element id near the top of the script.
2. **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`). Read through `localStorage` at startup and written on change.
3. **Tuning data**: `DIFFICULTIES` (spawn rates, enemy speed, shot chance, starting lives and bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palette per stage, cycled), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses with colors and HP multipliers).
4. **Audio**: lazily created `AudioContext` (`ensureAudio`), plus `beep`, `noiseBurst`, and the `sfx` table. Audio is created on first user input because browsers require a gesture.
5. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
6. **Layout**: `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to fit the window, capped at 1.6×.
7. **Input**: keyboard (`keys` map), mouse (steer and click-to-fire on the canvas), and touch (`joystick*`, `fireBtn`, `bombBtn`).
8. **Game state**: `resetGame()` initializes the player, arrays (`bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`), score, chain and multiplier, stage, and boss.
9. **Gameplay**: `spawnEnemy`, `spawnBoss`, `useBomb`, `fireWeapon` (Vulcan and Laser, each with power levels), `update(dt)`, the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), and `updateBoss(dt)`.
10. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
11. **Loop**: `endGame()` records the best score and shows the overlay; `loop(now)` drives `update` and `draw` through `requestAnimationFrame`.

Key gameplay facts:
- Stages: each stage ends after `KILLS_PER_STAGE` (18) kills and spawns a boss. Stages cycle through `THEMES` and `BOSSES`.
- Power-up types: `life`, `bomb`, `weapon`.
- Scoring uses a kill-chain multiplier (`chain`, `chainTimer`, `multiplier`).
- Controls: mouse or arrows/WASD to move, Space or click to fire, X or Shift for a bomb, P or Esc to pause. The on-screen hint in `#controlsHint` documents these, so keep it in sync if controls change.
- Adaptive quality: `trackPerf` samples frame times and steps `quality` (0 full, 1 medium, 2 low) when frames run slow.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split. Do not add a build system, framework, or external dependencies without asking.
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const` by default, short descriptive function names, and brief `//` section headers such as `// ---------- Game state ----------`.
- Tuning values live in the data objects near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer editing those over hard-coding numbers in logic.
- Persist user settings only through the `skyfighter.*` localStorage keys. Keep the best-score key stable so existing players keep their records.
- Preserve accessibility hooks: the `srLive` live region (via `announce`), the reduced-motion setting (`reducedMotion`, which respects `prefers-reduced-motion`), and the mute toggle.
- Keep the layout usable at phone width. The page uses `touch-action: none` and fixed-viewport scaling, so avoid changes that allow page scrolling.

## Git workflow

- Develop on the branch assigned to the session. Do not push to other branches without explicit permission.
- Use clear, descriptive commit messages that explain the gameplay or UI change.
- Do not open a pull request unless the user asks for one.
