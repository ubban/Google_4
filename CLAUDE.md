# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser on a 2D canvas.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1670 lines).
- `README.md` — minimal placeholder; not a source of truth for the game.

There is no build step, package manager, bundler, framework, or test suite. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by loading the page and playing: start a game, check firing, bombs, pause, boss fights, and game over.
- For touch/mobile layout changes, check the `touchControls` overlay at phone widths.

## Code structure (inside `index.html`)

All game logic lives in a single IIFE (`(function () { 'use strict'; ... })();`). Relevant sections, roughly in file order:

| Area | Notes |
|---|---|
| DOM refs | `const xxxEl = document.getElementById(...)` block at the top of the script. Keep IDs in the markup and these refs in sync. |
| Persistence | `STORAGE_*` keys prefixed `skyfighter.` (best score, mute, difficulty, reduced motion). Accessed via `localStorage`. |
| Adaptive quality | `quality` (0 full / 1 medium / 2 low) adjusted by `trackPerf` from a rolling frame-time average. |
| Config tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`. Tune gameplay here rather than hard-coding numbers in logic. |
| Audio | Web Audio via `ensureAudio()`, `beep()`, `noiseBurst()`. Audio starts only after a user gesture. |
| Input | Keyboard (`keys` map, `keydown`/`keyup`), mouse/pointer steering via `canvasPointFromEvent`, and touch joystick + FIRE/BOMB buttons. |
| Game state | `resetGame()` initializes state; `update(dt)` advances simulation; `draw()` renders; `loop(now)` drives both via `requestAnimationFrame`. |
| Weapons & power-ups | `fireWeapon()`, `applyPowerUp()`, and the `fire*` pattern helpers used by bosses. |
| Bosses | `spawnBoss()`, `updateBoss()`, `killBoss()`, and `BOSS_PATTERNS`. |
| Accessibility | `announce()` writes to the `srLive` live region; `reducedMotion` disables screen shake, flashes, and pulses. |

## Conventions

- Keep the game in `index.html` unless the user asks for a split. Do not introduce external dependencies or CDN scripts.
- Use `const` by default; the existing code uses `let` only for mutable state.
- Match the existing style: 2-space indentation, single quotes, arrow functions for callbacks, and section comments in the form `// ---------- Name ----------`.
- Time-based logic uses `dt` in seconds (clamped to 0.05s in the loop). Do not tie movement or timers to frame counts.
- Guard every `localStorage` access, since it can throw in private or restricted contexts. Current code does not wrap it, so add try/catch when touching it.
- Keep gameplay tuning in the config tables above rather than scattering magic numbers.

## Git

- Develop on the branch the session specifies (currently `claude/gifted-tesla-xbouua`). Do not push to other branches without permission.
- Do not open a pull request unless explicitly asked.
