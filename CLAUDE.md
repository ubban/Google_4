# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game. The entire game (markup, CSS, and JavaScript) lives in one file, `index.html` (~58 KB). There is no build step, package manager, bundler, test suite, or server-side code.

Other files:
- `README.md` — a one-line placeholder title.
- `.git/` — the repository; work is on the `claude/gifted-tesla-d1orrn` branch.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static file server (for example `python3 -m http.server`). No install step is needed.

## File structure of `index.html`

- **Lines ~1–362 — `<style>`**: dark color scheme (`color-scheme: dark`), HUD pills, vignette/flash overlays, difficulty buttons, touch controls, animations (`heartBeat`, etc.).
- **Lines ~363–421 — markup**: `<canvas id="gameCanvas">`, HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar), start/pause/game-over overlay, difficulty buttons (`.diffBtn[data-diff]`), mute and motion toggles, mobile touch controls (move zone, joystick, fire, bomb).
- **Lines ~422–1665 — one IIFE `<script>`** wrapped in `(function () { 'use strict'; ... })();`. Nothing is exposed on `window`. Sections, roughly in order:
  - DOM references and `STORAGE_*` keys
  - `announce()` (screen-reader live region) and `trackPerf()` (adaptive performance)
  - `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and other tuning tables
  - Audio: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` helpers (WebAudio, muted via a toggle)
  - HUD helpers: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`
  - Input: keyboard (`keydown`/`keyup` on `keys[e.code]`), mouse on the canvas, touch joystick and buttons
  - Game lifecycle: `resetGame()`, `update(dt)`, `draw()`, `loop(now)`, `endGame()`, `togglePause()`, `loseLife()`
  - Entities and drawing: `drawPlane`, `drawBossShip`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`
  - Weapons and bosses: `fireWeapon()`, `useBomb()`, `applyPowerUp()`, boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `updateBoss`, `killBoss`
  - Scoring: `addScore()`, `breakChain()` (kill-chain multiplier)
  - `resize()` and the start-button handler; `requestAnimationFrame(loop)` kicks off the loop at the end

## Game design facts (for code changes)

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy shoot chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills and leads to a boss fight. Bosses are picked from `BOSSES` by `(stage - 1) % BOSSES.length`, and their HP scales with stage. Backgrounds cycle through `THEMES`.
- **Weapons**: Vulcan and Laser, each with power levels 1–3, upgraded by power-ups.
- **Bombs**: limited smart bombs that clear the screen and damage bosses.
- **Power-ups**: weapon level, bomb, extra life.

## Controls

- Mouse: move to steer, click to fire.
- Keyboard: Arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Touch: drag the left zone to fly, tap FIRE, tap BOMB.

## Persistence

Stored in `localStorage` with these keys (all prefixed `skyfighter.`):
- `bestScore`
- `muted` (`'1'` = muted)
- `difficulty`
- `reducedMotion` (`'1'` / `'0'`; if unset, the default is taken from the OS `prefers-reduced-motion` setting)

Any change to these keys is a change to saved player data. Keep them stable, or add a migration.

## Conventions and cautions

- **Keep everything in `index.html`.** The project is intentionally dependency-free and single-file. Do not add a build system, framework, or external CDN dependency without a clear reason.
- **Preserve the IIFE.** Do not leak globals; the file relies on `'use strict'` and closure-scoped state.
- **Time-based updates.** `update(dt)` takes a delta time; tune gameplay values in the tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in update logic.
- **Accessibility.** Respect `reducedMotion` for screen-shake and flash effects. Announce important state changes through `announce()`, which feeds the `srLive` live region.
- **Mobile.** The layout uses `viewport-fit=cover`, safe-area insets, and `touch-action: none`. Test touch controls when changing input or layout code.
- **Audio.** All sound goes through the WebAudio helpers and must be initialized from a user gesture via `ensureAudio()`. Respect the mute toggle.
- **Performance.** `trackPerf()` adapts effects to frame time. Keep per-frame allocations low in `update`/`draw`.

## Verification

There are no automated tests or linters. To verify a change:
1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Play through start, a stage, a boss fight, pause/resume, bomb use, and game over.
3. Check difficulty switching, mute, and reduced-motion toggles persist across reloads.
4. For layout or input changes, check a narrow mobile viewport with touch emulation.

## Git

- Develop on the feature branch the task names (currently `claude/gifted-tesla-d1orrn`); do not push to other branches without permission.
- Write clear, descriptive commit messages.
