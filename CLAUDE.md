# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-page browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`. The whole game lives in one file, `index.html` (inline CSS, inline JavaScript, no external dependencies). `README.md` is a placeholder.

There is no build step, package manager, bundler, linter, or test suite. To run the game, open `index.html` in a browser.

## Layout

- `index.html` — everything:
  - `<style>` (around lines 10–361): layout, HUD, overlay, touch controls, `.reduced-motion` styles.
  - `<body>` markup (around lines 362–421): `#container` with the `#gameCanvas`, HUD (`#hud`, `#bossBarWrap`, `#topBar`), `#overlay` (start / pause / game-over card), and `#touchControls` (joystick and fire/bomb buttons).
  - One `<script>` (around lines 422–1668) wrapped in an IIFE with `'use strict'`. Nothing is exported to `window`.
- `README.md` — project title only.
- `.git` — the repo. Work happens on feature branches; do not commit directly to `main`.

## Architecture of the script

The script is organized in sections marked by `// ---------- ... ----------` comments and grouped function declarations:

- **DOM references and constants** — elements are cached at the top (`canvas`, `ctx`, HUD elements, buttons). Storage keys use the `skyfighter.*` prefix.
- **Config tables**
  - `DIFFICULTIES` — `easy` / `normal` / `hard`, controlling spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
  - `THEMES` — six sky palettes cycled per stage (`(stage - 1) % THEMES.length`).
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2).
  - `BOSSES` — six bosses, each with colours and an `hpMul`. Each boss has an attack list in `BOSS_PATTERNS` (radial, aimed fan, columns, homing, cross, double-ring bullet patterns).
  - `KILLS_PER_STAGE = 18`, which triggers a boss fight.
- **Audio** — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object). No audio files. `ensureAudio()` runs on the first user gesture.
- **Adaptive quality** — `trackPerf` samples the last 45 frame times and steps `quality` (0 full, 1 medium, 2 low) to keep frame time near 60 fps. Explosion particle counts scale with it, and shadow glow is disabled at lower levels.
- **Input** — keyboard (`keydown`/`keyup` on `window`, stored in `keys[e.code]`), touch joystick and buttons, and pointer/canvas handling via `canvasPointFromEvent`.
- **Game state** — module-level `let`s in the IIFE: `score`, `lives`, `maxLives`, `bombCount`, `stage`, `stageKills`, `player`, `enemies`, `bullets`, `enemyBullets`, `powerUps`, `boss`, `running`, `paused`, `gameOver`. `resetGame()` reinitialises them.
- **Simulation** — `update(dt)` advances everything: player movement and firing (`fireWeapon`), spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), collisions (`rectsOverlap`), scoring with a combo multiplier (`addScore`, `breakChain`), bombs (`useBomb`), power-up effects (`applyPowerUp`), and boss behaviour (`updateBoss`, `killBoss`).
- **Lifecycle** — `loseLife()` (with invulnerability frames), `endGame()` (updates `bestScore` in `localStorage`), `togglePause()`.
- **Rendering** — `draw()` renders the frame. Helpers include `drawPlane`, `drawBossShip`, and `drawClouds`. `resize()` scales the fixed `BASE_W × BASE_H` (480×720) logical canvas to fit the viewport.
- **Main loop** — `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw` while running and unpaused. It starts with `requestAnimationFrame(loop)`.

## Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Joystick (left) |
| Fire (hold) | Space | Fire button |
| Bomb | X or Shift | Bomb button |
| Pause / resume | P or Escape | Pause button |

Mute, reduced-motion, and pause buttons sit in `#topBar`. Difficulty is chosen from the start overlay (`.diffBtn`).

## Persistence

Stored in `localStorage` under these keys (all prefixed `skyfighter.`):

- `bestScore`
- `muted`
- `difficulty`
- `reducedMotion` (if unset, falls back to `prefers-reduced-motion`)

Keep new persisted settings under the same prefix.

## Conventions

- Keep the game self-contained in `index.html`. Do not add a build step, framework, or external CDN dependency unless the user asks for it.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, section banner comments (`// ---------- Name ----------`), and short explanatory comments only where the logic is not obvious.
- Gameplay numbers (speeds, HP, spawn rates, scoring) live in the config tables or near their use. Tune them there rather than adding new magic numbers in the update loop.
- Accessibility features (the `#srLive` live region via `announce()`, the `reduced-motion` body class, and `title` text on icon buttons) should be preserved and extended when adding new UI or events. Icon buttons should keep a `title`, and new icon-only controls should get an `aria-label` too.
- Respect the `quality` level when adding particles or glow effects.

## Verifying changes

There are no automated tests. To check a change:

1. Open `index.html` in a browser (or serve the directory with any static server).
2. Play through at least one stage and one boss fight on each difficulty you touched.
3. Check the touch layout by enabling device emulation, and the reduced-motion path via the in-game toggle.
4. Confirm the browser console shows no errors.

Syntax can be checked quickly with `node --check` on the extracted script if needed.
