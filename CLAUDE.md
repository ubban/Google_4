# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up browser game.

- `index.html` (~1,670 lines) is the entire game: markup, CSS (`<style>`), and JavaScript (one IIFE in a `<script>` block). The game renders to a `<canvas id="gameCanvas">` using the 2D context.
- `README.md` is a placeholder and contains no setup instructions.
- There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not invent `npm` / test commands. Open `index.html` directly in a browser, or serve the directory with any static file server, to run the game.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The whole game (HTML, CSS, JS) |
| `README.md` | Placeholder readme |
| `CLAUDE.md` | This file |

## Code structure of `index.html`

Within the main IIFE, code is organized into commented sections (search for these headers):

- **Setup / DOM refs**: `canvas`, overlay, HUD elements, and buttons are pulled from `document.getElementById` at the top.
- **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`) in `localStorage`.
- **Adaptive performance** (`trackPerf`): rolling average of frame times over 45 frames drives a `quality` level (0 full, 1 medium, 2 low).
- **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palette), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six bosses: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN).
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `sfx`): Web Audio synthesis only; no audio files. Mute state is persisted.
- **HUD and feedback**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, and `announce` (writes to the `#srLive` ARIA live region).
- **Input**: keyboard handler on `window`, a virtual joystick and fire/bomb buttons for touch, and mouse handling on the canvas.
- **Game state**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` / `applyPowerUp`, `addScore` / `breakChain` (score multiplier combo), `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`.
- **Boss attacks**: `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` and `killBoss` manage phases.
- **Rendering**: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`, plus `resize()` which scales a fixed `BASE_W = 480`, `BASE_H = 720` playfield to the viewport.
- **Main loop**: `loop(now)` clamps `dt` to 0.05 s, calls `trackPerf`, then `update` and `draw` when running and not paused.

## Controls

- Move: arrow keys or `WASD`; touch uses the on-screen joystick.
- Fire: `Space` (hold), or hold the fire button / mouse button.
- Bomb: `X` or `Shift`.
- Pause: `P` or `Escape`.

## Conventions

- Keep the game self-contained in `index.html` unless the user asks for a split. Do not add build tooling, frameworks, or external dependencies without asking.
- Preserve the existing style: `const`/`let`, arrow functions for callbacks, two-space indentation, and section comments with `// ---------- Name ----------` headers.
- Tuning values live in the constant tables near the top of the script. Prefer changing those over hard-coding numbers in logic.
- Persisted settings use the `skyfighter.` key prefix. Keep new keys under that prefix and wrap storage access in a way that tolerates unavailable storage if you touch it.
- Accessibility features exist and should be kept working: reduced-motion toggle (`body.reduced-motion` CSS overrides), the `aria-live` announcer, and touch controls alongside keyboard/mouse.
- Respect `prefers-reduced-motion` when adding screen shake, flashes, or pulses. Add a matching `body.reduced-motion` override in the CSS.

## Verifying changes

There is no automated test harness. To verify a change:

1. Open `index.html` in a browser (or serve the folder statically) and check the console for errors.
2. Play through: start, move, fire, collect a power-up, use a bomb, take damage, pause/unpause, reach a stage transition, and fight a boss.
3. Check both keyboard and touch layouts if the change touches input or HUD.
4. Toggle mute, reduced motion, and difficulty, and confirm they persist across reloads.

Report in your summary which of these you actually ran. Be clear when verification was done only by reading the code.

## Git

- Development branch for this work: `claude/gifted-tesla-93wyat`. Commit and push there; do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Keep commit messages short and imperative, describing the gameplay or code change.
