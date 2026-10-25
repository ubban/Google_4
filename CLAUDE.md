# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–360), and the game logic (one inline `<script>` IIFE, ~lines 422–1669).
- `README.md` — a placeholder title only.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Making changes

- Keep the game self-contained in `index.html` unless the user asks for a split. Do not introduce external libraries or CDN scripts.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for callbacks, and `// ---------- Section ----------` banner comments to separate logical areas.
- Game tuning values live in data tables near the top of the script. Prefer editing those over hard-coding numbers in logic:
  - `DIFFICULTIES` — spawn rates, enemy shoot chance, lives, bombs, boss HP multiplier per difficulty.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — sky gradient, cloud/sun colors, and names for each stage's background.
  - `ENEMY_TYPES` — size, HP, speed, colors, score, and the stage each enemy first appears in.
  - `BOSSES` — boss names, colors, and HP multiplier. `BOSS_PATTERNS` holds the bullet patterns.
- Persistent state uses `localStorage` with keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Keep that prefix for any new key, and guard reads and writes if you add new ones.
- Accessibility features are part of the game, so keep them working when you touch nearby code: the `aria-live` region (`srLive`) fed by `announce()`, the reduced-motion toggle (which also honors `prefers-reduced-motion`), and the keyboard controls.
- The adaptive quality tracker (`trackPerf`, `quality` 0–2) measures frame times over 45 frames. Keep frame-heavy effects gated on it when adding new visuals.

## Game architecture (inside the `<script>`)

- **Loop:** `requestAnimationFrame(loop)` computes `dt` (capped at 0.05 s), then calls `update(dt)` and `draw()` while the game is running and not paused.
- **Update / draw split:** `update()` handles movement, firing, collisions, spawning, and scoring. `draw()` handles all rendering. `drawPlane`, `drawBossShip`, and `drawClouds` are the drawing helpers.
- **Enemies and bosses:** `spawnEnemy`, `spawnBoss`, `updateBoss`, and `killBoss`. Bosses cycle through `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Player systems:** weapons (`fireWeapon`, `weaponLabel`), power-ups (`maybeSpawnPowerUp`, `applyPowerUp`), a score combo chain (`addScore`, `breakChain`), bombs (`useBomb`), and lives (`loseLife`). Game over runs through `endGame`.
- **Audio:** Web Audio API, created lazily by `ensureAudio()` on the first user gesture. Sounds come from `beep()` and `noiseBurst()`. Respect the `muted` flag. Do not start audio before a user gesture.
- **Input:** keyboard (arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause), mouse (move to steer, click to fire), and touch (a left-side joystick, plus FIRE and BOMB buttons). Touch handlers call `preventDefault` and are non-passive.
- **Layout:** `resize()` scales a fixed logical canvas of `BASE_W` × `BASE_H` (480 × 720) to fit the window, including on orientation change.

## Verifying changes

No automated tests exist. After editing, verify by hand:

1. Open `index.html` in a browser and check the browser console for errors.
2. Play through: start, move, fire, collect a power-up, use a bomb, pause and resume, lose a life, and reach game over.
3. Check that the best score, mute, difficulty, and reduced-motion settings persist after a reload.
4. Check the layout at phone width and in landscape, and test touch controls if you changed input code.
5. If you changed a boss or stage, play until it appears (the `KILLS_PER_STAGE` threshold controls when stages advance).

When you can't run the game in the environment, say so and describe what you verified by reading the code.

## Git and pull requests

- Develop on the branch the session names. Commit with clear messages that describe the change, and push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
