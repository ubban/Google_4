# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up (Raiden-style) built on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup, CSS (`<style>` block), and JavaScript (`<script>` block, wrapped in an IIFE with `'use strict'`).
- `README.md` — minimal project readme (currently just a title).

There is no build step, package manager, bundler, test suite, or linter configured. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture notes (`index.html`)

- **Canvas and scaling:** the logical play field is `BASE_W = 480` × `BASE_H = 720`; `resize()` scales the canvas to the window. Gameplay coordinates should stay in logical units.
- **Main loop:** a single `requestAnimationFrame` loop (`loop(now)`) calls `update(dt)` then `draw()` while the game is running and not paused. `dt` is clamped to 0.05s to avoid large jumps after tab switches.
- **Difficulty:** the `DIFFICULTIES` table (`easy` / `normal` / `hard`) controls spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. Add new tuning values there rather than hard-coding them.
- **Stages and bosses:** `KILLS_PER_STAGE` sets progression; `THEMES` defines the per-stage sky palette; `BOSSES` and `BOSS_PATTERNS` define boss bodies and attack patterns (radial, spread, spiral, cross, homing, etc.).
- **Weapons and power-ups:** Vulcan and Laser weapons with 3 power levels (`fireWeapon`, `applyPowerUp`). Power-ups include weapon level, bomb, and extra life.
- **Audio:** synthesized with the Web Audio API (`beep`, `noiseBurst`, `ensureAudio`). Audio is created lazily on the first user gesture, so keep it that way to satisfy browser autoplay policies.
- **Persistence:** `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write should tolerate storage being unavailable (private mode, blocked storage).
- **Accessibility:** a `reducedMotion` setting disables screen shake and flashes; `announce()` writes to a live region (`srLive`) for screen readers. Respect both when adding new visual effects.
- **Input:**
  - Keyboard: arrow keys / WASD-style movement via `keys[e.code]`, `Space` to fire, `X` or `Shift` for a bomb, `P` or `Escape` to pause.
  - Mouse/pointer: canvas pointer events via `canvasPointFromEvent`.
  - Touch: on-screen joystick (`moveZone`), fire button, and bomb button, with `passive: false` listeners where `preventDefault` is needed.

## Conventions

- Keep the game self-contained in `index.html` unless there is a clear reason to split it. If it is split, update this file.
- Do not add external CDN dependencies, fonts, or images without a reason; the game currently needs none.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in JS, small named helper functions, and section grouping by comment.
- Name new `localStorage` keys with the `skyfighter.` prefix.
- Any new tunable number should go in an existing config table (`DIFFICULTIES`, `BOSSES`, `ENEMY_TYPES`, `THEMES`) when it fits there.
- Keep the game playable on both desktop and touch devices when changing input or layout.

## Verification

Since there are no automated tests, verify changes by:

1. Loading `index.html` in a browser with no console errors.
2. Playing through at least one stage transition and one boss fight after gameplay changes.
3. Checking pause/resume, bomb use, mute, difficulty switch, and best-score persistence after reloading.
4. Checking the touch controls in a mobile-sized viewport when input or layout code changes.

A headless browser (Playwright/Chromium) can be used for smoke tests if available.

## Git

- Default work branch for this checkout: `claude/gifted-tesla-okozp7`.
- Write clear commit messages describing the gameplay or code change.
- Do not commit local-only files or secrets.
