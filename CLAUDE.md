# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. The whole game is a single self-contained file:

- `index.html` (~1,670 lines): inline `<style>`, markup, and one IIFE `<script>` containing all game logic, rendering (2D canvas), audio (Web Audio API), input, and UI.
- `README.md`: placeholder only.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` and visit `/index.html`). There are no dependencies to install.

## Architecture (inside `index.html`)

The script is wrapped in an IIFE with `'use strict'`. Sections, in source order, are marked with `// ---------- Name ----------` comments:

- **DOM refs** — `document.getElementById` lookups at the top of the script.
- **Constants** — `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
- **Adaptive performance** — `trackPerf(dt)` averages the last 45 frame times and steps a `quality` level (0 full / 1 medium / 2 low) up or down. Effects should respect this level when adding cost.
- **Audio** — `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture). SFX go through `beep`, `noiseBurst`, and named entries in `sfx`. Mute state persists in localStorage.
- **Input** — keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and touch (virtual joystick on the left, FIRE and BOMB buttons).
- **Sizing** — fixed logical resolution `BASE_W × BASE_H` = 480 × 720; `resize()` scales the canvas CSS size to fit the window (capped at 1.6×).
- **Game state** — `resetGame()` initializes state; `update(dt)` advances the simulation; `draw()` renders. `loop(now)` drives both via `requestAnimationFrame`, clamps `dt` to 0.05 s, and only updates/draws when `running && !paused`.
- **Weapons and bombs** — `fireWeapon()`, `applyPowerUp()`, `useBomb()`.
- **Bosses** — `spawnBoss()`, `updateBoss(dt)`, `killBoss()`, with patterns in `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Lifecycle** — `loseLife()`, `endGame()`, `togglePause()`.

Persistence uses `localStorage` with keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should tolerate storage being unavailable.

## Controls (keep the on-screen hint in sync)

- Move: mouse, Arrow keys, or WASD. Fire: Space or mouse click/hold. Bomb: X or Shift.
- Pause: P or Esc.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

The controls hint text lives in `#controlsHint` (`index.html` near line 417). Update it whenever bindings change.

## Conventions

- **Keep it a single file.** New code goes in `index.html` unless the user asks for a split. Don't introduce frameworks or external CDN dependencies.
- **Match existing style:** 2-space indent, single quotes in JS, `const` by default, section comments in the `// ---------- Name ----------` format, and tuning values as named constants in the tables near the top of the script rather than magic numbers scattered through logic.
- **Accessibility:** announcements go through `announce(text)`, which writes to the `#srLive` region. Respect `reducedMotion` for shake and flash effects (see `screenShake`, `flashHit`, `flashBomb`).
- **Mobile:** the layout must work at phone width and handle touch. Test touch changes with touch emulation, not only the mouse.
- **Difficulty tuning:** change values in `DIFFICULTIES`, `ENEMY_TYPES`, and `BOSSES` instead of hard-coding them in `update`/`spawn*` functions.

## Verification

No automated tests exist. To verify a change:

1. Load `index.html` in a browser (or a headless Chromium via Playwright) and check the browser console for errors.
2. Play through start → a few kills → a bomb → pause/resume → game over, on at least one difficulty.
3. For UI or layout changes, check both a desktop viewport and a phone-sized viewport.

If you add tests or tooling, document the commands here.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-9w5nmr`). Do not push to other branches without permission.
- Don't open pull requests unless explicitly asked.
