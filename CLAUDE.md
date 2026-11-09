# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up
built with HTML5 Canvas and vanilla JavaScript. There is no build step, no
package manager, and no test suite.

- `index.html` — the entire game: markup, CSS (`<style>` at ~line 10), canvas
  element (~line 363), and game logic (`<script>` at ~line 422).
- `README.md` — minimal placeholder.

## Running the game

Open `index.html` directly in a browser. No server or install is required.

## Code layout (inside `index.html`)

All JavaScript lives in one IIFE (`(function () { 'use strict'; ... })();`).
Sections, roughly in file order:

| Section | Purpose |
| --- | --- |
| DOM refs | `getElementById` lookups for HUD, overlay, touch controls |
| Persistence | `localStorage` keys prefixed `skyfighter.` (best score, mute, difficulty, reduced motion) |
| Adaptive performance | `trackPerf` lowers `quality` (0/1/2) when frame time is high |
| Config tables | `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Audio | Web Audio helpers: `ensureAudio`, `beep`, `noiseBurst`, `sfx.*` |
| HUD/FX | hearts, bombs, combo pops, stage banner, screen shake, flashes |
| Input | keyboard, touch joystick + fire/bomb buttons, mouse on canvas |
| Game state | `resetGame`, `update(dt)`, `draw()`, `loop(now)` |
| Weapons/bosses | `fireWeapon`, `applyPowerUp`, `BOSS_PATTERNS` (radial, spread, spiral, homing, etc.), `updateBoss`, `killBoss` |
| Flow | `loseLife`, `endGame`, `togglePause`, start button handler |

The main loop is `requestAnimationFrame(loop)`; `update(dt)` runs game logic and
`draw()` renders each frame.

## Game rules and tuning

- Difficulty (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Controls spawn
  rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP.
- Stages advance every `KILLS_PER_STAGE` (18) kills and end in a boss from `BOSSES`.
- Enemies: `grunt` (stage 1+), `interceptor` (stage 1+, weaving), `tank` (stage 2+, 4 HP).
- Power-ups: `life`, `bomb`, `weapon` (weapon level up).
- Score is multiplied by a kill-chain combo (`addScore`, `breakChain`).

Prefer changing numbers in the config tables above rather than hard-coding them
in logic.

## Controls

- Move: arrow keys or WASD; touch joystick on mobile.
- Fire: Space (hold), or fire button on touch.
- Bomb: X or Shift, or bomb button.
- Pause: P or Escape.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add a
  build system or dependencies without asking.
- Keep the code style already in the file: 2-space indent, single quotes,
  `const`/`let`, arrow functions for callbacks, and short section banners
  (`// ---------- Name ----------`).
- Stay compatible with mobile widths and touch input when changing controls or HUD.
- Respect the reduced-motion setting when adding screen shake or flashes
  (`reducedMotion`).
- Use `announce(text)` for any new state change that should be read by screen readers.
- Persisted values go through the `STORAGE_*` constants, not new ad-hoc keys.

## Verification

There are no automated tests. To verify a change:

1. Open `index.html` in a browser and play through start, a stage transition,
   a boss fight, pause/resume, and game over.
2. Check the console for errors.
3. For HUD or layout changes, check at a phone-sized viewport as well as desktop.

## Git

- Development branch for this work: `claude/gifted-tesla-yrxg5u`.
- Commit with clear messages and push with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.
