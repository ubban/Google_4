# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only substantial content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-file HTML5 canvas game.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript in one file (~1,670 lines). |
| `README.md` | Minimal placeholder; currently just the repo title. |
| `CLAUDE.md` | This file. |

There is no build system, package manager, test suite, or linter configured. Nothing needs to be installed.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit `http://localhost:8000`.
- Works on desktop and mobile. Touch controls appear automatically on touch devices.

## Architecture of `index.html`

The file is structured in three blocks:

1. **`<style>`** (around lines 10–360): all CSS. The HUD is pill-style with heart lives and bomb counters; the overlay is a glassmorphic card. Responsive layout and reduced-motion support are handled here.
2. **`<body>` markup** (around lines 360–421): HUD elements, overlay/menu, touch controls, and a `<canvas id="gameCanvas">`. Element IDs are referenced directly by the script.
3. **`<script>`** (around lines 422–1668): one IIFE (`'use strict'`) containing all game logic. Nothing is exported to `window`.

Within the script, the main sections, in rough order:

- **Config constants**: `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs).
- **Persistence**: `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every read/write must tolerate the value being absent.
- **Adaptive quality**: `trackPerf` samples frame times and steps a `quality` level (0–2) up or down to hold roughly 40–56 fps.
- **Audio**: lazily created `AudioContext` (`ensureAudio`) using oscillator `beep` and `noiseBurst` helpers. Audio starts only after a user gesture.
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse, and touch (virtual joystick, FIRE, BOMB buttons).
- **Game state & loop**: `resetGame`, `update(dt)`, `draw()`, and a `requestAnimationFrame` `loop`. `dt` is clamped to 50 ms so pauses and tab switches don't cause large jumps.
- **Weapons & power-ups**: Vulcan/Laser with three levels, screen-clearing smart bombs, and power-ups (weapon level, bomb, extra life).
- **Bosses**: `spawnBoss`, `updateBoss`, `killBoss`, and a table of attack patterns in `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Scoring**: `addScore` with a kill-chain multiplier that `breakChain` resets.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, parallax layers, and `screenShake`/`flashHit` effects.
- **Accessibility**: an `aria-live` region (`announce`) for screen readers and a reduced-motion toggle.

### Controls (from the in-game hint)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mobile: drag left side to fly, tap FIRE, tap BOMB.

## Conventions

- **Keep it a single file** unless the user asks for a split. Preserve the self-contained, no-dependency design; do not add external libraries or CDN links without asking.
- **Stay in the IIFE.** New globals should not leak onto `window`.
- **Name storage keys with the `skyfighter.` prefix** and guard reads with defaults.
- **Use `dt`-based movement** (seconds, multiplied by speed) rather than per-frame increments, so behavior stays consistent across frame rates.
- **Tune gameplay through the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in logic.
- **Match the existing style**: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, short section banner comments (`// ---------- Name ----------`).
- Keep the reduced-motion and `aria-live` behavior working when adding visual effects or announcements.

## Verifying changes

There is no automated test suite. To check a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Exercise the affected path: start a game, pause/resume, use a bomb, reach a stage transition or boss, and game over, then confirm the best score persists after reload.
3. For visual changes, check both desktop and a phone-sized viewport, and with reduced motion on.
4. For audio changes, confirm mute persists across reloads and that no sound plays before the first user interaction.

If a headless browser is needed, Playwright with the pre-installed Chromium works well for screenshots and smoke tests.

## Git

- Development branch for this session: `claude/gifted-tesla-8mhuz5`.
- Commit messages in this repo are descriptive, with a short summary line followed by a body explaining the change (see `git log`).
- Do not open pull requests unless explicitly asked.
