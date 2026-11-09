# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project containing one game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — minimal placeholder.

There is no build step, no package manifest (`package.json`), no bundler, no framework, no linter config, and no automated test suite. Do not introduce any of these unless the user asks.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, e.g. `python3 -m http.server` and visit `http://localhost:8000`. No install step is needed.

## Layout of `index.html`

| Section | Approx. lines | Contents |
|---|---|---|
| `<head>` | 1–9 | Meta tags, viewport, theme color, inline SVG favicon |
| `<style>` | 10–359 | All CSS, including the reduced-motion and mobile touch-control styles |
| `<body>` markup | 361–421 | `#gameCanvas`, HUD (score, best, multiplier, weapon, hearts, bombs), overlay/menu, difficulty buttons, touch controls, boss bar, screen-reader live region |
| `<script>` | 422–1667 | One IIFE (`'use strict'`) containing all game logic |

Inside the script, the code is organized by `// ----------` section comments:

- **Setup & DOM refs** — cached element lookups and `STORAGE_*` localStorage keys.
- **Adaptive performance** — `trackPerf` samples frame times and steps `quality` (0 full / 1 medium / 2 low) to shed effects on slow devices.
- **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt / interceptor / tank), `BOSSES` (six bosses, cycled by stage), `BOSS_PATTERNS`.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and a `sfx` object. Audio is synthesized with WebAudio; there are no audio asset files.
- **Rendering helpers** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw`.
- **Gameplay** — `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `update`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
- **Input** — keyboard (`keydown`/`keyup` on `window`), mouse on the canvas, and touch joystick plus FIRE/BOMB buttons.
- **Main loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` only when `running && !paused`.

## Game rules worth knowing

- **Controls:** Mouse moves and clicks to fire; Arrows/WASD move; Space fires; X or Shift drops a bomb; P or Esc pauses. Touch: drag left side to fly, tap FIRE, tap BOMB.
- **Weapons:** `vulcan` and `laser`, each with levels 1–3. A weapon power-up raises the level; past level 3 it swaps weapon type and resets to level 1.
- **Power-ups:** `life`, `bomb`, `weapon`, dropped at random from destroyed enemies (`maybeSpawnPowerUp`).
- **Score:** Base points per enemy type, multiplied by a combo chain (`addScore`, `breakChain`).
- **Stages:** Every `KILLS_PER_STAGE` (18) kills triggers a boss. Stage also selects the theme.
- **Persistence:** Best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under `skyfighter.*` keys. Code that reads them must tolerate missing or invalid values.

## Conventions

- **Keep it one file.** Put new CSS in `<style>`, new JS in the existing IIFE. Splitting into modules or external files is a larger change that needs the user's go-ahead.
- **No dependencies.** Use vanilla JS and the Canvas 2D / WebAudio APIs already in use. Do not add CDN scripts without asking.
- **Preserve the IIFE and `'use strict'`.** Game state is module-scoped with `let`/`const`; avoid adding globals.
- **Tune through the tables.** Balance changes belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, and `THEMES` rather than in scattered literals.
- **Respect the accessibility hooks.** Changes to visual effects should honor `reducedMotion` and the `quality` level, and any new user-facing status change should be announced through `announce()` for screen readers.
- **Keep the mobile layout working.** The viewport is non-scalable and the canvas is scaled to fit (`resize()`, base 480×720). Check touch controls when changing HUD or layout CSS.
- **Frame-rate independence.** Gameplay math uses `dt` in seconds. Keep new movement and timers in that unit.

## Verifying changes

There is no test runner or lint command. For changes to game behavior or layout:

1. Load `index.html` in a browser and play through at least one stage and one boss.
2. Check the three difficulty settings if the change touches balance.
3. Check a narrow/mobile viewport if the change touches layout or touch controls.
4. Check the browser console for errors.

If a headless browser is available (e.g. Playwright with Chromium), it can be used for scripted checks; do not add a test framework to the repo without asking.

## Git

- Development happens on feature branches; do not push directly to `main`.
- Write clear commit messages describing the gameplay or code change.
