# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up, built as a single-page HTML5 canvas game. There is no build step, package manager, framework, or test suite.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–360), and JavaScript (`<script>`, starting ~line 422). Roughly 1,670 lines.
- `README.md` — a one-line title. Keep it minimal.
- `CLAUDE.md` — this file.

## Running the game

No install or build is needed. Open `index.html` directly in a browser, or serve the directory with any static file server (for example `python3 -m http.server`). Keyboard, mouse, and touch controls are all supported.

## Verifying changes

There are no automated tests, linters, or CI configured. To verify a change:

1. Load `index.html` in a browser and check the browser console for errors.
2. Exercise the affected path: start a game, fire, use a bomb (`X`/`Shift`), pause (`P`/`Esc`), and let the game reach the relevant stage or boss if you changed it.
3. For UI or layout changes, check both a desktop viewport and a phone-width viewport, since touch controls appear on touch devices.

If you add a test or build tool, document the command here.

## Code architecture (`index.html`)

All game logic lives in one IIFE (`(function () { 'use strict'; ... })();`) with no globals. Keep new code inside that scope.

Major sections, in file order:

- **DOM references** — `const xEl = document.getElementById(...)` block at the top of the script.
- **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Changing a key orphans saved player data.
- **Adaptive quality** — `trackPerf()` samples frame times and steps `quality` (0 full, 1 medium, 2 low) to lower visual cost on slow devices.
- **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes, cycled by stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`, with `minStage` gating), and `BOSSES` (six bosses with colors and `hpMul`). Most balancing changes belong here.
- **Audio** — Web Audio API. `ensureAudio()` creates the context lazily on the first user gesture. Sound effects use `beep()` and `noiseBurst()`. Respect the `muted` flag.
- **Input** — keyboard (`keys` map via `e.code`), mouse (`canvasPointFromEvent`), a virtual joystick and fire/bomb buttons for touch (`joystickStart/Move/End`). Touch controls are enabled only on touch devices.
- **Game state** — module-level `let` variables (`score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, etc.), reset by `resetGame()`.
- **Spawning and entities** — `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()`. Enemy and bullet entities are plain objects in arrays (`enemies`, `bullets`, `enemyBullets`, `powerUps`).
- **Weapons and bombs** — `fireWeapon()` (Vulcan/Laser, 3 power levels), `useBomb()` (clears the screen, damages bosses).
- **Boss patterns** — `BOSS_PATTERNS` is an array of functions that push bullets into `enemyBullets`; `updateBoss()` cycles through them.
- **Update / draw** — `update(dt)` advances simulation with a delta time clamped to 50 ms; `draw()` renders the frame. `loop(now)` is the `requestAnimationFrame` driver. Simulation runs only when `running && !paused`.
- **Lifecycle** — `loseLife()`, `endGame()`, `togglePause()`, `killBoss()`.

## Conventions

- **Single file.** Do not split the game into multiple files, add a bundler, or add dependencies unless the user asks. The no-build design is intentional.
- **Frame-rate independence.** Movement and timers use `dt` in seconds. Follow that pattern rather than per-frame increments.
- **Pure canvas rendering.** Draw with `ctx` primitives. Do not introduce DOM-based sprites for game entities. HUD elements (hearts, bombs, score, banners) are DOM nodes and update through their existing helpers (`renderHearts`, `renderBombs`, `showStageBanner`, `showComboPop`).
- **Accessibility.** Announce important state changes through `announce()`, which feeds the `srLive` live region. Honor `reducedMotion` for screen shake and flashes.
- **Keep the existing style.** Two-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks. Match the surrounding code rather than reformatting unrelated lines.
- **Tuning changes.** Prefer editing the constant tables over hard-coding numbers in logic.

## Git and workflow

- Default branch: `main`. Feature work for this session goes on `claude/gifted-tesla-hzvduj`.
- Commit messages in this repo use a short imperative subject followed by a body describing the change in prose.
- Do not open a pull request unless the user asks for one.
- Do not commit generated or scratch files. The repo currently tracks only `README.md`, `index.html`, and this file.
