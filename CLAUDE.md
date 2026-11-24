# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up.

- `index.html` is the entire game: inline `<style>`, markup, and one inline `<script>` IIFE (roughly lines 10 / 361 / 422 / 1669).
- `README.md` is a placeholder stub (a title and a timestamp). Do not treat it as documentation.
- There is no build step, package manager, bundler, test suite, linter, or CI config. Nothing needs installing.

To run the game, open `index.html` in a browser. It loads no external scripts, stylesheets, or fonts; everything is inline, including the favicon (a data URI).

## Architecture of `index.html`

All game code lives in one `(function () { 'use strict'; ... })()` block. Key parts, in file order:

- **DOM references**: `const` lookups by element id at the top of the script (`gameCanvas`, `overlay`, HUD elements, touch controls).
- **Tunables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six boss definitions with colors and HP multipliers).
  - `BOSS_PATTERNS`: bullet-pattern generators for bosses.
- **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access is a direct call; new code should wrap them in try/catch in case storage is blocked.
- **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`). No audio files. Audio starts on first user gesture.
- **Rendering**: a 2D canvas, logical size `BASE_W × BASE_H` (480×720), scaled by `resize()`. Drawing helpers: `drawPlane`, `drawBossShip`, `drawClouds`, all called from `draw()`.
- **Simulation**: `update(dt)` advances the world each frame. `updateBoss(dt)` handles boss phases. `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, and `applyPowerUp` manage entities. Collisions use `rectsOverlap`.
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse/pointer (steer and fire), and touch joystick plus FIRE and BOMB buttons.
- **Loop**: `loop(now)` runs under `requestAnimationFrame`. `dt` is clamped to 0.05 s. It calls `update` and `draw` only while `running && !paused`.
- **Accessibility**: `announce()` writes to a live region. `reducedMotion` disables screen shake and flashes. `prefers-reduced-motion` seeds the default.

### Controls (as shipped)

| Action | Keys / input |
|---|---|
| Move | Arrow keys or WASD; mouse; touch drag (left zone) |
| Fire | Space (held) or click / FIRE button |
| Bomb | X or Shift, or BOMB button |
| Pause | P or Esc |
| Mute / motion toggles | On-screen buttons |

## Conventions

- Keep the game in the single file unless the user asks for a split. Do not add a bundler, framework, or npm dependency without explicit approval.
- Stay in vanilla JS and ES2017+ syntax that runs in current evergreen browsers. Match the surrounding style: 2-space indent, single quotes, semicolons, `const` by default.
- New tunable numbers belong in the existing config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`), not scattered as magic numbers.
- Preserve the `'use strict'` IIFE. Do not leak globals.
- Any new visual or audio effect must respect `reducedMotion` where it involves shake, flashing, or large motion.
- Keep touch and keyboard controls working together. Test both when changing input.

## Verification

No automated tests exist. Before reporting a gameplay or rendering change as working:

1. Open `index.html` in a browser (or serve the directory with any static server).
2. Check the browser console for errors on load and during play.
3. Exercise the changed path: start a game, change difficulty, trigger the affected enemy, boss, or power-up, and confirm pause, mute, and game over still work.
4. For layout changes, check a narrow (phone-width) viewport as well as desktop.

If you cannot run the game in a browser, say so explicitly rather than claiming it works.

## Git

- `main` is the base branch. Work for this session goes on `claude/gifted-tesla-r0q8ni`.
- Commit messages: short imperative summary (for example, `Add Sky Fighter: ...`).
- Do not create pull requests unless the user asks for one.
- Do not commit secrets or local settings files.
