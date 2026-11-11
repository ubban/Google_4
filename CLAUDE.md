# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: markup (`<div id="container">`, HUD, overlay, touch controls), inline `<style>`, and one inline `<script>` (roughly lines 422–1667).
- `README.md` — minimal project title only.
- No build step, no package manager, no dependencies, no test suite. Open `index.html` in a browser to run it.

## Running and verifying changes

- Run: open `index.html` directly, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, ideally at both desktop and phone-sized viewports (the game supports touch controls).
- Keep it a single self-contained HTML file unless the user asks to split it up.

## Code layout inside the script

All game state and logic live in one IIFE-style scope. Key sections, in file order:

| Area | Notes |
| --- | --- |
| DOM lookups (`~L426`) | Cached element references (`canvas`, HUD ids, overlay, touch buttons). |
| Storage keys (`~L463`) | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`. |
| Config tables (`~L485–515`) | `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (6 stage backgrounds, cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (6 bosses, indexed by `(stage - 1) % 6`). |
| Audio (`~L543–680`) | Lazy `AudioContext` via `ensureAudio()`; `beep`/`noiseBurst` helpers and the `sfx` map. Mute state persists. |
| Input (`~L768–860`) | Keyboard (`keys`), touch joystick + FIRE/BOMB buttons, and mouse steering (`mouseTarget`). |
| Game state (`~L864–910`) | `resetGame()` initializes `player`, arrays (`bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`), `boss`, score, lives, bombs, stage, chain/multiplier. |
| Drawing (`~L913–1020`, `~L1507–1640`) | `drawPlane`, `drawBossShip`, and `draw()`. Canvas is fixed at 480×720 (`BASE_W`/`BASE_H`) and scaled by `resize()`. |
| Spawning (`~L1022–1068`) | `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()` (power-up types: `life`, `bomb`, `weapon`). |
| Update loop (`~L1148–1303`) | `update(dt)` — movement, firing, collisions, scoring, stage progression. |
| Power-ups / weapons (`~L1124`, `~L1304`) | `fireWeapon()`, `applyPowerUp()`. |
| Boss patterns (`~L1324–1410`) | `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss()`. |
| Lifecycle (`~L1441–1500`) | `killBoss()`, `loseLife()`, `endGame()`, `togglePause()`. Best score is written to `localStorage` on game over. |
| Main loop (`~L1640`) | `loop(now)` drives `update` and `draw` via `requestAnimationFrame`, with a `quality` setting (full/medium/low) adjusted by `trackPerf()` based on frame times. |

## Gameplay constants to know

- Difficulty (`easy` / `normal` / `hard`) changes spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. Selected via the overlay buttons (`.diffBtn`) and persisted.
- Lives and bombs are capped by difficulty (`maxLives`, `maxBombs`).
- Score uses a chain/multiplier system (`chain`, `chainTimer`, `multiplier`, `breakChain()`).
- Accessibility: a reduced-motion toggle (screen shake, flashes, heart pulse) defaults from `prefers-reduced-motion` and is persisted.

## Conventions

- Plain vanilla JS and CSS; no frameworks, no transpilation. Keep the file loadable by opening it directly.
- Preserve the existing style: 2-space indentation, `const`/`let`, small helper functions, config-as-object tables for enemies/bosses/difficulty.
- The existing `localStorage` reads/writes are not wrapped in `try/catch`. New storage access should be wrapped so the game still runs when storage is blocked (private windows, disabled site data).
- Keep touch and keyboard/mouse controls in parity when adding actions.
- Keep HUD and overlay text in sync with the controls hint (`#controlsHint`).

## Git workflow

- Development branch for this task: `claude/gifted-tesla-n4bz7e`. Base history: `main`.
- Do not open a pull request unless explicitly asked.
