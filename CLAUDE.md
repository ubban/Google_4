# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`, with inline CSS and one inline `<script>` block. There is no build step, no package manager, no bundler, and no external dependencies.

`README.md` is a placeholder and does not describe the game. Treat `index.html` as the source of truth.

## Repository layout

- `index.html` — the whole game (~1,670 lines): markup, styles, and game logic.
- `README.md` — placeholder.
- `CLAUDE.md` — this file.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There is no test suite, linter, or build. Verify changes by loading the page and playing: check keyboard, mouse, and touch controls, and confirm the HUD, audio, and game-over flow still work.
- For a visual or behavioral check, a headless Chromium with Playwright is fine. Do not install a new browser; use the pre-installed one.

## Code structure (inside `index.html`)

Game logic lives in one IIFE-style script. Key sections, in rough file order:

- **Constants and config** (~line 463–516): `STORAGE_*` localStorage keys, `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss definitions).
- **Persistent settings**: best score, mute, difficulty, and reduced-motion are stored in `localStorage`. Reads and writes are wrapped by the code that uses them; keep that pattern.
- **Audio**: Web Audio API. `ensureAudio()` lazily creates the context (browsers require a user gesture), with `beep()` and `noiseBurst()` as the primitives. Sound effect names are mapped in `EXPLODE_SFX`.
- **UI/HUD**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, and `announce` (screen-reader live region).
- **Layout and input**: `BASE_W = 480`, `BASE_H = 720` define the logical play area; `resize()` scales the canvas. Input is handled for keyboard (`keys` map; P/Escape pause, X/Shift bomb), a virtual joystick and fire/bomb buttons for touch, and mouse on the canvas.
- **Game state**: top-level `let` variables (`score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, etc.). `resetGame()` reinitializes them.
- **Drawing**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. Drawing is procedural canvas 2D; there are no image assets.
- **Update logic**: `update(dt)` runs movement, collisions, spawning, and scoring. Spawning is in `spawnEnemy` / `spawnBoss`; power-ups via `maybeSpawnPowerUp` and `applyPowerUp`.
- **Weapons and boss patterns**: `fireWeapon()` for the player; `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing` for boss attacks.
- **Main loop**: `loop(now)` drives `requestAnimationFrame`. `trackPerf(dt)` adjusts a `quality` level (0 full, 1 medium, 2 low) when frame times are slow; respect this when adding expensive effects.

## Conventions

- **Keep it a single file.** Do not split `index.html` into modules, add a build step, or add dependencies unless the user asks. The no-build, open-and-play property is a deliberate feature.
- **Match the existing style**: two-space indentation inside the `<script>`, `const` for config tables, `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constants, and `skyfighter.*` prefixes for storage keys.
- **Respect the performance budget.** Gate new particle or visual effects on `quality` and `reducedMotion`. Avoid per-frame allocations in hot paths where practical.
- **Keep both input modes working.** Any control change must work on keyboard, mouse, and touch. Touch targets are used on mobile; `touch-action: none` is set deliberately.
- **Accessibility**: use `announce()` for important state changes that screen readers should hear, and honor the reduced-motion setting for shake, flash, and pulse animations.
- **Tuning changes go in the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than being hard-coded into logic.

## Git workflow

- Develop on the branch designated for the session (currently `claude/gifted-tesla-zqu0aq`). Do not push to `main` without explicit permission.
- Write clear, descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless the user asks for one.
