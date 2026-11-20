# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), canvas element, and all JavaScript (one IIFE `<script>`, ~lines 422–1668).
- `README.md` — minimal placeholder; no setup instructions.

There is no build step, package manager, test suite, or linter. Nothing to install.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- Verify gameplay changes in a real browser, not only by reading the code. The Playwright/Chromium setup in the environment is sufficient for this.
- Test both keyboard and touch paths when touching input code; touch controls are shown only on touch-capable devices (`isTouchDevice`).

## Code layout inside `index.html`

The script is wrapped in `(function () { 'use strict'; ... })();`. Everything is private; do not add globals.

Major sections, in file order:

| Area | Key identifiers |
|---|---|
| DOM refs | `canvas`, `ctx`, overlay/HUD element lookups at top of script |
| Storage keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*`) |
| Adaptive quality | `quality` (0/1/2), `trackPerf()` — lowers effects detail when frame time is high |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx.*` — WebAudio, created lazily on first user gesture |
| HUD | `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `announce()` (screen-reader live region) |
| Input | `keys` map, `joystick*` functions, `canvasPointFromEvent`, touch/keyboard listeners |
| Game state | `resetGame()`, `player`, `bullets`, enemies, power-ups, `update(dt)` |
| Weapons & bosses | `fireWeapon()`, `applyPowerUp()`, pattern helpers `fireRadial`/`fireAimedSpread`/…, `BOSS_PATTERNS`, `updateBoss()`, `killBoss()` |
| Rendering | `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`, `screenShake` |
| Loop | `loop(now)` — fixed-clamped `dt` (max 50 ms), calls `update` then `draw` |

## Controls (for reference when editing input)

- Move: arrow keys / WASD via `keys[e.code]`, or touch joystick.
- Fire: Space (`e.preventDefault()` is called on Space to stop page scroll), or touch fire button.
- Bomb: `X` or either Shift, or touch bomb button.
- Pause: `P` or `Escape`.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce a bundler, framework, or npm dependencies without asking.
- **Use `const`/`let`**, arrow functions for callbacks, and keep the existing 2-space indentation and single-quoted strings.
- **Data-driven tuning.** Enemy, boss, difficulty, and theme numbers live in the constant tables near the top of the script. Prefer adding or adjusting entries there over hard-coding values in logic.
- **Time is in seconds.** Movement and timers are `dt`-scaled; keep new motion frame-rate independent.
- **Persist only through the `skyfighter.*` localStorage keys.** Wrap new storage reads/writes if they could be called in restricted contexts, and add a new `STORAGE_*` constant rather than an inline string.
- **Accessibility.** Respect `reducedMotion` (body class `reduced-motion`) for new visual effects that shake, flash, or move heavily. Announce important state changes through `announce()`.
- **Audio** must start only after a user gesture; use `ensureAudio()` before playing sound.
- **Performance.** The game runs on a `requestAnimationFrame` loop. Avoid per-frame allocations in hot paths (`update`, `draw`) where practical, and gate expensive effects on `quality`.

## Git

- Development branch for this session: `claude/gifted-tesla-55qfy8`. Commit and push there; do not push to other branches without permission.
- Do not open a pull request unless the user asks for one.
