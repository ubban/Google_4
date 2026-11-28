# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up with weapon switching, smart bombs, stages, and bosses.

The whole game lives in one file, `index.html`:

- `<style>` (around lines 10–361): CSS for the page, HUD, overlays, and touch controls.
- `<body>` markup (around lines 363–421): the `<canvas id="gameCanvas">`, HUD elements, overlays, and touch controls.
- `<script>` (around lines 422 onward): a single IIFE with all game logic, rendering, audio, and input.

`README.md` contains only a title. There is no build system, package manager, bundler, or test suite.

## Running it

- No install or build step. Open `index.html` directly in a browser, or serve the directory with any static file server (for example, `python3 -m http.server`).
- Verify changes by loading the game in a browser and playing it. There are no automated tests to run.

## Code structure (inside the `<script>`)

Roughly in order of appearance:

- **Config constants**: `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses with colors and HP multipliers), and `BOSS_PATTERNS` (bullet patterns).
- **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should keep working if storage is unavailable.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and `panFor`, built on the Web Audio API. Audio starts only after a user gesture and can be muted.
- **Input**: keyboard (`keys` map, Space fire, X/Shift bomb, P/Esc pause), mouse/pointer steering and click to fire, and touch joystick plus FIRE/BOMB buttons.
- **Game loop**: `update(dt)` handles simulation (spawning, collisions, power-ups, boss logic), `draw()` handles rendering, and `loop(now)` drives both via `requestAnimationFrame`.
- **Game state**: `resetGame`, `loseLife`, `endGame`, `killBoss`, `togglePause`. Stage progression is driven by kill count.
- **Rendering helpers**: `drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion`, `screenShake`, and the HUD helpers (`renderHearts`, `renderBombs`, `showStageBanner`, `showComboPop`).
- **Canvas sizing**: `resize()` fits a fixed logical canvas (`BASE_W` × `BASE_H` = 480 × 720) to the viewport.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add a build step, framework, or dependencies without asking.
- Match the existing style: vanilla JS inside the IIFE, `const`/`let`, camelCase identifiers, `UPPER_SNAKE` for module-level constants, and single quotes.
- Keep the game's accessibility features working: `prefers-reduced-motion` handling (`reducedMotion`), the `aria-live` announcer (`announce` / `srLive`), and the touch controls.
- Tuning values (difficulty tables, enemy and boss stats) live in the config constants at the top of the script. Prefer changing those over hard-coding numbers in logic.
- Keep the page usable on phones: the layout uses viewport meta settings, `touch-action: none`, and safe-area insets, so preserve them when editing CSS.

## Git

- Development branch for this session: `claude/gifted-tesla-ghubl6`. Commit and push there. Do not push to other branches without explicit permission.
- Do not open pull requests unless the user asks.
