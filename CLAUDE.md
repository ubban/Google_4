# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a single-file, Raiden-style vertical shoot-'em-up that runs in the browser. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` IIFE (roughly lines 422–1669).
- `README.md` — minimal placeholder.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). Nothing needs to be installed.

## Architecture (inside `index.html`)

The script is wrapped in `(function () { 'use strict'; ... })()` and organized by section comments (`// ---------- Input ----------`, etc.). Key pieces:

- **Canvas and scaling** — fixed logical resolution `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas CSS size to fit the viewport, capped at 1.6×. Game logic and drawing always use the base coordinates.
- **Difficulty** — `DIFFICULTIES` (`easy` / `normal` / `hard`) controls spawn rates, enemy speed, enemy shot chance, starting lives, starting bombs, and boss HP multiplier.
- **Enemies** — `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), each gated by `minStage`. `spawnEnemy()` picks from the types available at the current stage.
- **Bosses** — `BOSSES` array cycled by `(stage - 1) % BOSSES.length`. `spawnBoss()` sets HP from `70 + stage * 35` scaled by the boss and difficulty multipliers. Boss attack patterns are the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring) driven by `updateBoss()`.
- **Stages** — `KILLS_PER_STAGE = 18` kills trigger the boss warning; `killBoss()` increments `stage`. `THEMES` holds the background palettes.
- **Weapons and power-ups** — the player has `vulcan` or `laser` with a `level`. Power-ups (`life`, `bomb`, `weapon`) are dropped by `maybeSpawnPowerUp()` and applied by `applyPowerUp()`.
- **Update/draw loop** — `update(dt)` handles simulation, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`.
- **Audio** — synthesized with the Web Audio API (`beep`, `noiseBurst`, the `sfx` table). Audio is created lazily on first user gesture via `ensureAudio()`.
- **Input** — keyboard (arrows/WASD move, Space fire, X or Shift bomb, P/Esc pause), mouse (move to steer, click to fire), and touch (joystick zone on the left, FIRE and BOMB buttons).
- **Persistence** — `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. All reads and writes go through this key set; keep the prefix when adding new ones.
- **Accessibility** — `announce()` writes to the `#srLive` live region. A reduced-motion toggle (also seeded from `prefers-reduced-motion`) adds `body.reduced-motion`, which turns off the CSS animations (heartbeat, stage banner, vignette, flashes), and `flashHit()`/`flashBomb()` return early when it is set.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. Do not introduce a bundler, framework, or npm dependency without asking.
- Use `const`/`let`, keep the `'use strict'` IIFE wrapper, and match the existing 2-space indentation.
- Game tuning values live in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) — prefer editing those over hard-coding numbers in logic.
- Time-based motion uses `dt`; do not tie movement to frame count.
- New HUD elements need matching CSS and should respect the `reduced-motion` class and phone-width layout (the existing HUD uses `clamp()` sizing and `safe-area-inset` padding).
- Every new storage key uses the `skyfighter.` prefix.

## Verification

There are no automated tests or lint scripts. To verify a change:

1. Load `index.html` in a browser and play through at least one stage transition and one boss fight.
2. Check the console for errors.
3. Check the mobile layout at a narrow viewport and confirm touch controls still respond.
4. For changes to difficulty or spawn logic, try all three difficulty settings.

## Git

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-nofjaj`). Do not push to other branches without permission.
- Write commits with clear, descriptive messages.
