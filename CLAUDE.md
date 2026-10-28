# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single-file HTML5 canvas game.

- `index.html` (~1,670 lines): the entire game. CSS is in a `<style>` block (starts around line 10), markup is in the body (the `<canvas id="gameCanvas">` is around line 363), and all game logic is in one inline `<script>` (starts around line 422).
- `README.md`: a one-line title. It is not a full project description.

There is no build step, no bundler, no package manager, no test suite, and no linter configuration. Do not add them unless the user asks.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Because there are no tests, verify gameplay changes by running the game in a browser and exercising the affected path (movement, firing, bombs, stage transitions, boss fights, pause, game over).
- For mobile-facing changes, check the touch controls and the layout at phone width. The page sets `viewport-fit=cover` and uses touch joystick and fire/bomb buttons.

## Code layout inside `index.html`

The inline script is organized roughly in this order. Keep new code in the matching section.

1. **Persistence keys**: `STORAGE_*` constants prefixed `skyfighter.` (best score, mute, difficulty, reduced motion). Read and write them through `localStorage`.
2. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` map. Audio is created lazily on the first user gesture.
3. **Tuning data**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here, not in the update loop.
4. **HUD and feedback**: `renderHearts`, `renderBombs`, combo pops, stage banners, screen shake, hit and bomb flashes.
5. **Layout**: `BASE_W = 480`, `BASE_H = 720` define the logical play field. `resize()` scales it to the window.
6. **Input**: keyboard (`keys` map), a touch joystick, fire and bomb buttons, and mouse aiming on the canvas.
7. **Game state**: `let` variables for score, lives, bombs, stage, chain and multiplier, plus `resetGame()`.
8. **Entities and weapons**: drawing helpers (`drawPlane`, `drawBossShip`), spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), and weapons (`fireWeapon`, `applyPowerUp`).
9. **Boss patterns**: `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireCross`, `fireHomingSeeded`, and others). `updateBoss` cycles through them.
10. **Main loop**: `update(dt)` runs game logic, `draw()` renders, and `loop(now)` is the `requestAnimationFrame` driver. `trackPerf` lowers the `quality` level (full, medium, low) when frame times stay high.

## Conventions

- Plain JavaScript with no modules, classes, or transpilation. Use `const` and `let`, and match the existing two-space indentation and single quotes.
- Keep the game in one file unless the user asks for a split. The existing layout is organized with comments and section order, not with files.
- Put tuning values in the data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in logic.
- Respect the reduced-motion setting. Visual effects that shake or flash should check `reducedMotion`.
- Respect the mute setting. Any new sound must go through `beep` or `noiseBurst`, which check `muted`.
- Keep the game playable at phone width and with touch input. New controls need a touch path as well as a keyboard path.
- Preserve the existing controls: arrow keys or WASD to move, Space to fire (mouse click or hold on the canvas, and the FIRE button on touch, also fire), `X` or Shift for a bomb, `P` or Escape to pause.
- If you change a control, update the `#controlsHint` text in `index.html` (around line 417) to match.

## Git workflow

- Develop on the branch specified by the session, not on the default branch.
- Write clear, descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless the user asks for one.
