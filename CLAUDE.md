# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 canvas game, **Sky Fighter**, a Raiden-style vertical shoot-em-up.

- `index.html` is the entire app: inline CSS (`<style>`, ~lines 10–360), the markup (`<body>`, ~line 361), and one inline `<script>` (~line 422 to end, ~1,670 lines total).
- `README.md` is a placeholder and has no setup instructions.
- There is no package.json, build step, bundler, test suite, or linter. Nothing needs to be installed.

## Running and verifying

- Run it by opening `index.html` in a browser, or by serving the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. Check at least one desktop viewport and one narrow or touch viewport, since the game has touch controls.

## Code layout inside the `<script>`

The script is one IIFE-style block with no modules. Major sections, roughly in file order:

| Area | What it holds |
| --- | --- |
| DOM refs | `const xxxEl = document.getElementById(...)` for every HUD, overlay, and control element |
| Storage keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`) |
| Perf | `trackPerf` adapts a `quality` level (0 full, 1 medium, 2 low) from average frame time |
| Config tables | `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE` |
| Audio | Lazy `ensureAudio()` on a Web Audio context; `beep`, `noiseBurst`, and the `sfx` table. Mute state is persisted in `STORAGE_MUTE` (`'1'` means muted; sound is on by default) |
| HUD and FX | `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit` |
| Layout | `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the window, capped at 1.6x |
| Input | Keyboard (`keys`), touch joystick and fire/bomb buttons, and mouse via `canvasPointFromEvent` |
| Entities | `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` |
| Game flow | `resetGame`, `spawnEnemy`, `spawnBoss`, `useBomb`, `fireWeapon`, `update(dt)`, `draw()`, `loop(now)` |

The main loop is `loop(now)` → `update(dt)` → `draw()`, driven by `requestAnimationFrame`.

## Game design facts

These values live in the config tables. Change them there, not in scattered literals.

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs, and a boss HP multiplier.
- **Enemies** (`ENEMY_TYPES`): `grunt` (stage 1+), `interceptor` (stage 1+, weaves), `tank` (stage 2+, 4 HP).
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills and then a boss fight starts. Each stage uses the next entry in `THEMES`, cycling.
- **Bosses** (`BOSSES`): six designs, each with its own colors and HP multiplier. `drawBossShip` renders them with telegraphing before attacks.
- **Power-ups**: `life`, `bomb`, and `weapon`. Weapons are Vulcan and Laser, each with three levels.
- **Score**: a kill-chain multiplier (`chain`, `chainTimer`, `multiplier`) increases the score per kill.
- **Persistence**: the best score is kept in `localStorage`. Every `localStorage` access must be wrapped in try/catch, because it can throw in private browsing or when storage is blocked.

## Conventions

- Keep the single-file structure. Do not split out modules, add a build step, or add dependencies unless the user asks for it.
- Match the existing style: 2-space indentation, `const` and `let` (no `var`), single quotes, short helper functions, and brief comments only where the logic is not obvious.
- Put tunable numbers in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) instead of hard-coding them in logic.
- Gameplay code runs in frame-time `dt`. Keep movement and timers in `dt`-based units, not per-frame increments, so behavior stays consistent when the frame rate changes.
- Respect the reduced-motion setting (`reducedMotion`) and the quality level when adding visual effects. Heavy effects should degrade at `quality >= 1`.
- Any new user-visible state the game remembers goes under the `skyfighter.` storage prefix.
- Accessibility: new status changes should go through `announce()` (the `srLive` live region), and new controls need labels.

## Git

- Development branch for this work: `claude/gifted-tesla-qpfbgn`. Commit and push there; do not push to other branches without permission.
- Do not open a pull request unless asked.
