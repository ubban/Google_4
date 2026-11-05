# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game (markup, CSS, and JavaScript) lives in one file, `index.html`. There is no build step, no package manager, no bundler, and no test suite.

`README.md` is a one-line placeholder.

## Repository layout

- `index.html` — the whole game (~1,670 lines):
  - `<head>`: meta tags, an inline SVG favicon, and an inline `<style>` block (HUD pills, overlay card, touch controls, vignette/flash effects, keyframes).
  - `<body>` (around lines 362–421): `#container` holding the `#gameCanvas`, HUD (`#hud`), boss bar, combo/stage banners, top-bar buttons (mute, reduced motion, pause), touch controls, and the start/pause `#overlay`.
  - `<script>` (starts around line 422): one IIFE-style block with all game logic.
- `.git/` — git history. The default development branch for this task is `claude/gifted-tesla-aw1ih2`.

## Running and testing

- To run: open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests, linters, or build commands. Verify changes by loading the page in a browser and playing through the affected behavior. For desktop input, check mouse and keyboard; for mobile layout or touch changes, emulate a touch device.

## Code structure (inside the `<script>`)

Key sections, roughly in source order:

- **DOM refs and constants**: element lookups (`canvas`, `overlay`, `hud` nodes, touch buttons), `STORAGE_*` localStorage keys, and `BASE_W = 480`, `BASE_H = 720` (the logical playfield).
- **Tuning tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18) controls stage progression.
  - `THEMES`, `ENEMY_TYPES` (with `minStage` gating), and `BOSSES` (cycled by `(stage - 1) % BOSSES.length`).
- **Persistence**: best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under `skyfighter.*` keys. Every access should stay wrapped so the game still works if storage is unavailable.
- **Audio**: a lazily created Web Audio context (`ensureAudio`) with `beep` / `noiseBurst` primitives and a named `sfx` table. Audio starts only after a user gesture.
- **Rendering**: `drawPlane`, `drawBossShip`, and `draw()`, plus the `resize()` function that scales the canvas to fit the window (capped at 1.6x).
- **Input**: keyboard (`keys` map), touch joystick and buttons, and mouse steering/firing on the canvas.
- **Game state**: `resetGame()` initializes the module-level state (`player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`/`multiplier`, and so on).
- **Simulation**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `useBomb`, and the per-frame `update(dt)`.
- **Flow control**: `endGame()`, `togglePause()`, the `loop(now)` game loop driven by `requestAnimationFrame`, and `trackPerf` / `quality` (adaptive quality levels 0–2 when frame times slip).

## Controls (for reference when changing input code)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Escape to pause.
- Mouse: move to steer, click to fire.
- Touch: drag in the left zone to fly with the virtual joystick, tap FIRE, tap BOMB.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. Do not introduce a build tool or framework without asking.
- Follow the existing style: 2-space indentation, `const`/`let`, plain DOM APIs, and no external runtime dependencies.
- Keep the game playable with no network access. Do not add external CDN scripts or fonts.
- Respect the reduced-motion setting (`reducedMotion`) for any new screen shake, flashes, or pulsing effects.
- Keep the HUD and overlay usable at phone width; the layout is designed for touch as well as desktop.
- Preserve accessibility hooks such as the `#srLive` live region when changing announcements.

## Git workflow

- Develop on the branch assigned to the task (currently `claude/gifted-tesla-aw1ih2`).
- Commit with clear, descriptive messages, and push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
