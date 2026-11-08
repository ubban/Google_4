# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), and JavaScript (`<script>`, ~lines 422–1669). No build step, no dependencies, no external assets.
- `README.md` — one-line project title only.

There is no package manager, test suite, linter, or CI configured.

## How to run

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. No install or build is needed.

## Code layout (inside `index.html`)

The script is one IIFE (`'use strict'`) with these main sections, in roughly this order:

| Area | Key identifiers |
| --- | --- |
| DOM references | `canvas`, `ctx`, `overlay`, HUD elements (`scoreEl`, `heartsEl`, `bombsEl`, `bossBarFill`, …) |
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (`localStorage` keys prefixed `skyfighter.`) |
| Tuning data | `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()` — Web Audio synth, no sound files |
| HUD / feedback | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake` |
| Layout | `resize()` — canvas sized to a fixed `BASE_W × BASE_H` (480 × 720) logical space |
| Input | `keys` map, touch joystick (`joystickStart/Move/End`), `canvasPointFromEvent` |
| Game state | `resetGame()`, `update(dt)`, `applyPowerUp()`, `useBomb()`, `fireWeapon()`, `endGame()`, `togglePause()` |
| Spawning | `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()` |
| Bosses | `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring), `updateBoss()`, `killBoss()` |
| Rendering | `drawPlane()`, `drawBossShip()`, `drawClouds()`, `draw()` |
| Main loop | `loop(now)` via `requestAnimationFrame`; `dt` clamped to 0.05 s |

Keep new code in the matching section rather than creating parallel helpers.

## Controls

- Move: arrow keys or WASD (keyboard), or the on-screen joystick (touch devices).
- Fire: hold Space (or the fire button on touch).
- Bomb: `X` or Shift (or the bomb button on touch).
- Pause: `P` or `Escape`.

## Conventions

- **Single file.** Keep everything in `index.html` unless the user asks for a split. Don't add a bundler, framework, or npm dependency without being asked.
- **Vanilla JS, ES2015+ style** consistent with the existing code: `const`/`let`, arrow functions for callbacks, `function` declarations for game logic.
- **Frame-rate independence.** Movement and timers should use the `dt` passed into `update()`, not frame counts.
- **Logical coordinates.** Game objects are positioned in the 480 × 720 base space; the canvas is scaled in `resize()`. Don't hard-code device pixel sizes.
- **Tuning in data tables.** Difficulty numbers, enemy stats, and boss definitions live in the constant tables near the top of the script; prefer editing those over scattering magic numbers.
- **Persistence keys** use the `skyfighter.` prefix. Wrap `localStorage` access so the game still runs if storage is unavailable (private mode, blocked storage).
- **Accessibility hooks.** Respect the reduced-motion setting (`STORAGE_REDUCED` / `motionBtn`) when adding screen shake or flashes, and keep the `srLive` live region updated for important announcements via `announce()`.
- **Mobile.** Changes to input or layout must keep touch controls working; `touchControls` is shown only when a touch device is detected.

## Verifying changes

There are no automated tests. After editing:

1. Open `index.html` in a browser (or a static server) and check the browser console for errors.
2. Play through the start screen, a few waves, a stage transition, a boss fight, bomb use, pause/resume, and game over.
3. Confirm the best score persists across reloads.
4. If the change touches layout or input, check at a phone-sized viewport and with touch emulation.

## Git

- Default development branch for this environment: `claude/gifted-tesla-4jxkio`.
- Write clear, descriptive commit messages in the imperative mood.
