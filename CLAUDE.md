# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game is a single self-contained file:

- `index.html` (~1,670 lines) — markup, inline CSS (`<style>`, starts ~line 10), canvas element (`#gameCanvas`, ~line 363), and one inline `<script>` IIFE (starts ~line 422) containing all game logic, rendering, audio, and input.
- `README.md` — currently a stub (title only).

There is no build step, package manager, test suite, or linter configured.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser; for headless checks, Playwright with the pre-installed Chromium works (see the environment notes for the browser path).
- Keep the file loadable as-is. Do not introduce a bundler, framework, or external runtime dependency without an explicit request.

## Code layout inside `index.html`

The script is one `'use strict'` IIFE. Roughly in order:

1. **DOM references and constants** (~lines 422–461): element lookups, `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`).
2. **Tuning tables** (~lines 485–516): `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
3. **Persisted settings** (~lines 517–540) loaded from localStorage. Reads and writes are direct calls; only some are wrapped in try/catch, so new storage code should be guarded.
4. **Audio** (~lines 545–700): a Web Audio–based `beep` / `noiseBurst` synth. Audio is initialized lazily via `ensureAudio()` on first user gesture.
5. **HUD and UI effects**: hearts/bombs rendering, hit/bomb flashes, combo pops, stage banner, screen shake, `announce()` for screen readers.
6. **Input**: keyboard handlers (~line 769), on-screen joystick and fire/bomb buttons for touch, pointer handling.
7. **Game state**: `resetGame()`, `update(dt)` (the main simulation, ~line 1148), spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, `useBomb`), power-ups (`applyPowerUp`).
8. **Boss patterns**: `BOSS_PATTERNS` and the `fire*` helpers (~lines 1324–1410), `updateBoss`, `killBoss`.
9. **Lifecycle**: `loseLife`, `endGame`, `togglePause`.
10. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` (~line 1520).
11. **Main loop**: `loop(now)` driven by `requestAnimationFrame`. `dt` is measured per frame and clamped to 50 ms; `update` and `draw` run only while `running && !paused`.

Canvas is a fixed logical size `BASE_W × BASE_H` = 480 × 720, scaled to fit the viewport by `resize()`. Game coordinates should always be expressed in this logical space.

## Controls (for reference when changing input)

- Move: arrow keys or WASD. Fire: Space (held). Bomb: X or Shift. Pause: P or Escape.
- Touch: left-side virtual joystick; on-screen fire and bomb buttons.
- Difficulty is selected from the in-game buttons (`.diffBtn`) and persisted.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Preserve the single-file, no-dependency design.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, short helper functions, and UPPER_SNAKE_CASE for tuning constants.
- Put new balance numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, etc.) rather than hard-coding them in logic.
- Any new persisted setting needs a `skyfighter.*` localStorage key, and its reads and writes should be wrapped in try/catch, since storage can be unavailable (private mode, blocked site data).
- New user-facing status changes should also call `announce()` so screen-reader users get the update.
- Respect the `reducedMotion` setting when adding screen shake, flashes, or other motion effects.
- Keep touch controls working when adding new inputs; the game must remain playable on phones (`touch-action: none`, safe-area insets are already handled).

## Git

- Development branch for this work: `claude/gifted-tesla-94mpaa`. Push there; do not push to other branches without explicit permission.
- Do not create pull requests unless the user asks for one.
- Commit messages should be short imperative summaries with a body when the change needs explaining.
