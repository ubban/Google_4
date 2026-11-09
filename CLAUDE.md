# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- The entire game lives in a single file, `index.html` (~1,670 lines): inline `<style>`, the markup, and one inline `<script>` IIFE.
- `README.md` is a placeholder and has no build or setup instructions.
- There is no package.json, bundler, test suite, linter, or CI config. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Use the Playwright/Chromium setup in the environment for any automated check of the game. Verify visual or gameplay changes in a real browser, not only by reading the code.

## Code structure (`index.html`)

The script is one IIFE with `'use strict'`. Its sections, roughly in file order:

| Area | Key identifiers |
| --- | --- |
| DOM refs | `canvas`, `ctx`, `overlay`, `startBtn`, HUD elements, touch controls |
| Storage keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*` in `localStorage`) |
| Perf | `trackPerf()` adapts a `quality` level (0 full, 1 medium, 2 low) from average frame time |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx.*` (WebAudio, synthesized, no asset files) |
| UI feedback | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce()` (aria-live) |
| Input | keyboard (`keys`), touch joystick (`joystick*`), fire/bomb buttons, mouse via `canvasPointFromEvent` |
| Game state | `resetGame()`, `update(dt)`, `loseLife()`, `endGame()`, `togglePause()` |
| Entities/spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `rectsOverlap` |
| Weapons/bombs | `fireWeapon()`, `applyPowerUp()`, `useBomb()` |
| Boss attacks | `BOSS_PATTERNS` and the `fire*` pattern helpers (radial, spread, columns, spiral, cross, homing, double ring), `updateBoss`, `killBoss` |
| Rendering | `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`, `resize()` (fixed 480×720 logical canvas, scaled to fit) |
| Loop | `loop(now)` drives `update` and `draw` via `requestAnimationFrame`; `dt` is clamped to 0.05 s |

## Conventions

- **Keep it a single file** unless the user asks otherwise. Do not add a build step, framework, or external dependency without a clear reason.
- **Vanilla JS, ES2015+ syntax**, matching the existing style: `const`/`let`, arrow functions, 2-space indent, single quotes.
- **Game tuning lives in data objects** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `BOSS_PATTERNS`). Prefer adjusting those over hardcoding numbers in logic.
- **Time-based movement** uses `dt` (seconds). Do not tie speeds to frame count.
- **Persistence** uses `localStorage` with the `skyfighter.` prefix. Wrap new reads and writes so the game still works when storage is unavailable or blocked.
- **Accessibility**: respect `prefers-reduced-motion` (`reducedMotion`, which disables shake/flash/pulse). Announce important state changes through `announce()`. Keep keyboard, mouse, and touch controls working together.
- **Audio** is synthesized with WebAudio. Audio must not start before a user gesture; `ensureAudio()` is called from input handlers.
- **Mobile**: the game is designed to work on phone-sized screens. Check touch layouts when changing HUD or control elements.

## Making changes

- Read the relevant section of `index.html` before editing. Most changes touch only one or two of the sections above.
- After editing, load the page in a browser and play briefly. Check that the game starts, fires, takes damage, and reaches game over without console errors.
- Keep changes focused. Don't reformat unrelated code.

## Git

- Development branch for this task: `claude/gifted-tesla-vtf58h`. Commit and push there; don't push to `main` without explicit permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless the user asks for one.
