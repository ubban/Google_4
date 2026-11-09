# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free front-end project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 `<canvas>`.

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: inline CSS (`<style>`, roughly lines 10–362), canvas and HUD markup, and one inline `<script>` IIFE (starts ~line 422) containing all game logic. |
| `README.md` | Minimal project readme (title and a timestamp line). |

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not add tooling unless the user asks for it.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` and visit `http://localhost:8000`).
- There are no automated tests. Verify changes by playing the game in a browser, and check both desktop (keyboard) and narrow/mobile viewports (touch controls) when touching input or layout code.

## Architecture of `index.html`

The script is a single closure `(function () { ... })()`. Keep new code inside it so globals stay clean.

- **Constants and tuning** near the top of the script: `BASE_W = 480`, `BASE_H = 720` (logical canvas size, scaled by `resize()`), `KILLS_PER_STAGE`, `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`.
- **Persistence** uses `localStorage` with keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, and `reducedMotion`. Wrap storage access so the game still works if storage is unavailable.
- **Audio** is generated with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`). The `AudioContext` is created lazily on the first user gesture; keep it that way to satisfy browser autoplay rules.
- **Main loop**: `update(dt)` advances state, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`. Game logic should go in `update`, and rendering only in `draw` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
- **Adaptive quality**: `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) that reduces visual effects when the frame rate drops. New expensive effects should respect this level.
- **Entities** are plain arrays and objects managed inside `update`. Spawning helpers (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`) and collision via `rectsOverlap` are the main extension points.
- **Weapons and power-ups**: `fireWeapon` (Vulcan/Laser, three power levels), `applyPowerUp`, `useBomb` (limited smart bombs that also damage bosses), and `breakChain` / `addScore` for the kill-chain multiplier.
- **Bosses**: `updateBoss`, `killBoss`, and the attack pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) selected from `BOSS_PATTERNS`.
- **Lifecycle**: `resetGame`, `loseLife`, `endGame`, and `togglePause`.
- **HUD and feedback**: DOM elements for hearts, bombs, weapon, and multiplier are updated by `renderHearts`, `renderBombs`, and the `show*` / `flash*` / `screenShake` helpers. An `announce()` live region supports screen readers.
- **Input**:
  - Keyboard: arrow keys/WASD movement, `Space` to fire, `X` or `Shift` for bombs, `P` or `Escape` to pause (see `keydown` handler).
  - Touch: a virtual joystick (`joystickStart/Move/End`), a fire button, and a bomb button.
  - Pointer: `canvasPointFromEvent` maps client coordinates into logical canvas space.
- **Accessibility and comfort**: a reduced-motion setting disables or softens shake and flashes. Preserve that behavior for any new visual effect.

## Conventions

- Keep the project as one self-contained HTML file unless the user asks for a split. No external network dependencies: the favicon is an inline SVG data URI and fonts fall back to system stacks.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, and short descriptive helper names.
- Use the logical `BASE_W`/`BASE_H` coordinate space for all game positions and sizes. Do not use raw CSS pixels inside the game loop.
- When adding a tunable value (speed, spawn rate, damage, score), put it with the other constants rather than burying a magic number in the logic.
- Any new persistent setting should use the `skyfighter.` key prefix and be read defensively.
- Preserve the `touch-action: none` and `user-select: none` behavior on the game surface; they are needed for correct mobile play.

## Git workflow

- Default working branch for this task is the branch the session was started on (currently `claude/gifted-tesla-bn55gc`). Commit and push there; do not push to `main` unless explicitly told to.
- Write commit messages in the imperative mood, with a short subject line and a body that explains *why* when it is not obvious.
- Do not open a pull request unless the user explicitly asks for one.

## Keeping this file current

Update this CLAUDE.md when the structure changes: new files, a split of `index.html` into modules, a build or test setup, or a change to the input scheme or persistence keys. Keep it factual and short; describe what exists, not plans.
