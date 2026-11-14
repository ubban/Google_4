# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-file browser game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up), with no build step, no dependencies, and no tests.

- `index.html` — the entire game: inline CSS, DOM markup for the HUD and overlays, and one inline `<script>` IIFE containing all game logic.
- `README.md` — placeholder (currently only a title and a stray timestamp).

There is no `package.json`, bundler, linter, or test suite. Do not introduce one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No install step is needed.

## Code layout inside `index.html`

- **`<style>`** (top of file): dark color scheme, HUD pills (`#hud`), overlays, touch controls, and reduced-motion styles.
- **`<body>`**: `#gameCanvas` (base resolution 480×720), `#hud` (score, best, multiplier, weapon, hearts, bombs, boss bar), `#overlay` (title / game-over screen with difficulty buttons and `#startBtn`), `#touchControls`, and a `#srLive` live region for screen readers.
- **`<script>`**: one `'use strict'` IIFE. Key sections, in file order:
  - DOM refs and `STORAGE_*` keys (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`).
  - Tuning tables: `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
  - Audio: lazily created `AudioContext` via `ensureAudio()`, with `beep`/`noiseBurst` helpers and an `sfx` table.
  - Input: keyboard (`keys` map), mouse (`mouseTarget`, click to fire), touch joystick + FIRE/BOMB buttons.
  - Responsive scaling: `resize()` scales the canvas to fit, capped at 1.6×.
  - Game state: `resetGame()`, `update(dt)`, `draw()`, `loop(now)`; `loop` is driven by `requestAnimationFrame` and caps `dt` at 50 ms.
  - Gameplay: `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, `fireWeapon`, boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `applyPowerUp`, `useBomb`, `loseLife`, `endGame`, `togglePause`.
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.

Gameplay facts worth knowing before editing:

- Power-ups are `life`, `bomb`, and `weapon`. Weapon power-ups level the current weapon up to 3, then switch between `vulcan` and `laser`.
- A stage advances after `KILLS_PER_STAGE` kills; the next boss is then spawned after a 1.2 s delay. Bosses cycle through `BOSSES` by stage number (`(stage - 1) % BOSSES.length`), and boss HP scales with stage and difficulty.
- Difficulty controls spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- The best score is written to `localStorage` in `endGame()` when the final score beats the stored best.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Preserve the single-file, zero-dependency design.
- Keep tuning values in the existing constant tables rather than scattering magic numbers.
- Preserve accessibility hooks: keep `#srLive` announcements (`announce()`) for state changes, and honor `prefers-reduced-motion` through `reducedMotion`.
- Touch and mouse must both keep working. Changes to input code need checking on both paths.
- Wrap any new `localStorage` access so the game still works when storage is unavailable (private mode, blocked storage). The existing accessors do not guard this; new code should.
- Match the existing style: 2-space indentation, single quotes, `const`/`let`, and small named functions.

## Verifying changes

There are no automated tests. For changes, open the page in a browser and check:

1. Start, play, pause (`P`/`Esc`), and game over all work.
2. Each difficulty starts with the expected lives and bombs.
3. Power-ups, bombs, and boss transitions behave correctly after reaching `KILLS_PER_STAGE` kills.
4. Layout works at phone width and in landscape, and touch controls respond.
5. The browser console shows no errors.

For visual or layout changes, a screenshot at a desktop and a phone-sized viewport is the fastest check.

## Git

- Default development branch for this session: `claude/gifted-tesla-aue4ke`.
- Commit messages should describe the change in plain terms. Do not open pull requests unless explicitly asked.
