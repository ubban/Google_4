# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up in the style of Raiden. There is no build step, no bundler, no package manager, and no test suite.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–359), and JavaScript (`<script>`, ~lines 422–1667). |
| `README.md` | One-line placeholder. |

To run the game, open `index.html` in a browser. Nothing needs to be installed.

## Architecture of `index.html`

The game body is one IIFE (`(function () { 'use strict'; ... })();`). Everything is a closure-scoped `let`/`const`/`function`; nothing is exported to `window`.

Major sections, in file order:

1. **DOM references** – `getElementById` lookups for HUD, overlay, touch controls, and buttons.
2. **Persistence keys** – `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`).
3. **Config tables**
   - `DIFFICULTIES` – `easy` / `normal` / `hard` tuning: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` – kills needed to advance a stage.
   - `THEMES` – six sky palettes that cycle per stage.
   - `ENEMY_TYPES` – `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+).
   - `BOSSES` – six bosses (`VANGUARD` → `TITAN`), each with colors and an `hpMul`.
   - `BOSS_PATTERNS` – bullet-pattern functions (radial, aimed spread, spiral, cross, homing, double ring).
4. **Audio** – a lazily created `AudioContext` (`ensureAudio`) with synthesized `beep()` and `noiseBurst()` effects. Sound is muted via the `muted` flag.
5. **HUD and feedback** – hearts/bombs rendering, combo pop, stage banner, screen shake, screen flash, and `announce()` for an `aria-live` region.
6. **Layout and input** – `resize()` scales the canvas to a fixed `BASE_W × BASE_H` (480 × 720) logical size. Keyboard, mouse, and touch (virtual joystick + FIRE/BOMB buttons) feed into shared `keys`, `firing`, and pointer state.
7. **Game state** – `resetGame()` initializes `player`, `bullets`, `enemies`, `boss`, `powerUps`, etc.
8. **Simulation** – `update(dt)` advances everything per frame. Helpers: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `useBomb`, `endGame`, `togglePause`.
9. **Rendering** – `draw()` plus `drawPlane`, `drawBossShip`, `drawClouds`. Drawing is plain 2D canvas.
10. **Main loop** – `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw` while `running && !paused`.

### Key mechanisms

- **Fixed-step-ish timing:** all movement is `speed * dt`, with `dt` clamped to 0.05 s so a tab switch cannot teleport objects.
- **Adaptive quality:** `trackPerf()` keeps a rolling window of 45 frames and nudges `quality` (0–2) up when the average frame time is slow and down when it is fast.
- **Weapons:** two weapon types, `vulcan` and `laser`, each with levels 1–3 controlling spread. Power-ups are `life`, `bomb`, and `weapon`.
- **Scoring:** `addScore()` applies a combo multiplier that `breakChain()` resets when the player takes damage.
- **Best score:** written to `localStorage` on game over only when it is beaten.

## Conventions

- **Keep it a single file** unless the user asks for a split. Do not introduce a build system, framework, or npm dependency without an explicit request.
- **Vanilla JS, ES2015+ syntax** as already used: `const`/`let`, arrow functions, template literals. Match the surrounding style (2-space indent, single quotes, semicolons).
- **Preserve the IIFE and `'use strict'`.** New globals should not be added to `window`.
- **Config over magic numbers.** Tunable values belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES`, not inline.
- **Accessibility is intentional.** Respect `reducedMotion` for shakes, flashes, and pulses; keep `aria-live` announcements and the `prefers-reduced-motion` default; keep the on-screen controls and their `title`/`aria` labels.
- **Responsive and touch-first.** The viewport meta disables zoom. Any new UI must work at phone width and with touch input, not only the mouse/keyboard.
- **Persistence keys are a public contract.** Renaming a `skyfighter.*` key silently resets players' saved best score, mute, difficulty, and motion settings. Do not rename them without a migration.

## Working on the game

- **Verifying changes:** there are no automated tests. Open `index.html` in a browser (or serve the directory with any static server) and play through the affected path. For gameplay tuning, check Easy, Normal, and Hard, and at least one stage transition and one boss fight.
- **Check both input paths** after touching controls: keyboard (Arrows/WASD, Space, X/Shift, P/Esc), mouse, and touch.
- **Check the reduced-motion toggle** after touching effects.
- **Check the console** for errors after each change. Note that the `localStorage` reads at startup (~lines 517–523) are not wrapped in `try/catch`, so if storage is blocked the script will throw. Wrap them if you touch that code.
- **Do not commit** build output, screenshots, or editor files. Nothing in the repo is generated.

## Git

- Development branch for this work: `claude/gifted-tesla-0a5o5i`. Do not push to other branches without explicit permission.
- Write clear, imperative commit messages that describe the gameplay or code change, e.g. `Add boss pattern for double ring`.
- Do not open pull requests unless the user explicitly asks.
