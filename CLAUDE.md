# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-page HTML5 canvas vertical shoot-'em-up (Raiden-style). The entire game lives in one file:

- `index.html` — inline `<style>`, markup, and one IIFE `<script>` (roughly lines 422–1667). No build step, no dependencies, no package manager.
- `README.md` — minimal placeholder.

To run it, open `index.html` in a browser (or serve the directory with any static server). There are no tests, linters, or CI configured.

## Architecture of `index.html`

The game script is wrapped in `(function () { 'use strict'; ... })();` and organized by section comments:

- **DOM refs and constants** (top of script): element lookups, `STORAGE_*` keys, `DIFFICULTIES`, `THEMES` (six stage backdrops, cycled by stage), `ENEMY_TYPES` (grunt / interceptor / tank), `BOSSES` (six bosses, each with a `BOSS_PATTERNS` attack set), `KILLS_PER_STAGE` (18).
- **Persistence**: best score, mute, difficulty, and reduced-motion are saved to `localStorage` under the `skyfighter.*` keys. Reads and writes are direct `localStorage` calls; keep them guarded if you add new ones.
- **Audio**: Web Audio synthesized sound effects (`beep`, `noiseBurst`, `ensureAudio`). No audio files. `ensureAudio()` must run from a user gesture.
- **HUD / feedback**: pill-style HUD (score, best, multiplier, weapon, hearts, bombs), combo pops, stage banner, screen shake, hit/bomb flashes, low-health vignette. Screen-reader updates go through `announce()` into the `#srLive` region.
- **Adaptive quality**: `trackPerf()` samples frame time over 45 frames and steps a `quality` level (0–2) up or down to shed effects on slow devices.
- **Layout**: a fixed logical play field of `BASE_W × BASE_H` (480 × 720) scaled by `resize()`.
- **Input**: keyboard (`keys` map keyed by `e.code`), a touch joystick (`moveZone`), and fire/bomb buttons. Pointer and touch events use `passive: false` where `preventDefault` is needed.
- **Game state and logic**: `resetGame()`, `update(dt)`, `fireWeapon()` (Vulcan/Laser, 3 power levels each), `useBomb()`, `spawnEnemy()`, `spawnBoss()`, `applyPowerUp()`, `addScore()` with kill-chain multiplier (`breakChain()`), `loseLife()`, `endGame()`, `togglePause()`.
- **Bosses**: `BOSS_PATTERNS` holds one attack set per boss (radial, aimed spread, columns, spiral, cross, homing, double ring, etc.), selected by `updateBoss()`. `killBoss()` advances the stage.
- **Rendering**: `draw()` plus `drawPlane()`, `drawBossShip()`, `drawClouds()` (parallax layers). Drawing is canvas-only; no DOM per-frame.
- **Main loop**: `loop(now)` clamps `dt` to 0.05 s and runs `update` + `draw` only when `running && !paused`.

### Controls (for reference when editing input code)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrows / WASD | Left-side joystick |
| Fire (hold) | Space | Fire button |
| Bomb | X or Shift | Bomb button |
| Pause | P or Esc | — |

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a bundler, framework, or npm dependencies without asking.
- **No external network or CDN dependencies.** The page must work offline from a file URL.
- **Match the existing style:** 2-space indent, `const`/`let`, arrow functions for callbacks, named `function` declarations for game-logic helpers, and section banners like `// ---------- Input ----------`.
- **Keep the loop allocation-light.** Avoid creating new objects or closures per frame in `update`/`draw` where a pooled or reused structure already exists. Respect `quality` and `reducedMotion` when adding visual effects.
- **Preserve accessibility hooks.** New player-facing status changes should be announced through `announce()` where it fits, and new interactive controls need keyboard and touch parity.
- **Mobile first.** Changes to layout or controls must work at phone width with the `touch-action: none` and safe-area handling already in place.

## Working on the game

- Balance numbers live in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`). Prefer tuning there over hard-coding values in logic.
- When adding a boss pattern, add it to `BOSS_PATTERNS` and make sure the bullet count and speed stay within the existing performance envelope.
- When adding a power-up, handle it in `maybeSpawnPowerUp()` (spawn) and `applyPowerUp()` (effect), and update the HUD if it has a visible state.
- Test changes by opening `index.html` in a browser and playing through at least one stage transition and a boss fight. Check desktop keyboard and a touch-sized viewport (for example, Chrome DevTools device mode).

## Git

- Development branch for this session: `claude/gifted-tesla-jcvx03`. Do not push to other branches without explicit permission.
- Commit messages should be short imperative summaries, with a body for non-trivial changes. The existing history uses a single-line subject plus a bulleted body.
- Do not open a pull request unless the user explicitly asks for one.
