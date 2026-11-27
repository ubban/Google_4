# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-file HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–361), HTML overlay/HUD (~lines 362–421), and one IIFE `<script>` (~lines 422–end) containing all game logic.
- `README.md` — minimal placeholder; no build or setup docs yet.

There is no build system, package manager, test suite, or linter configured. No dependencies are loaded; everything is vanilla JS and CSS.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code structure (inside the `<script>` IIFE)

- **Config tables** near the top: `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
- **Persistence** via `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should be tolerant of storage being unavailable.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`), with synthesized `beep` / `noiseBurst` effects. Audio must only start after a user gesture.
- **Layout**: fixed logical resolution `BASE_W = 480`, `BASE_H = 720`, scaled to the viewport by `resize()`.
- **Input**: keyboard (arrows/WASD, Space fire, X/Shift bomb, P/Esc pause), mouse (steer and click to fire), and touch (joystick on the left, FIRE/BOMB buttons).
- **Game loop**: `update(dt)` mutates state, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`. `trackPerf` monitors frame timing.
- **Game flow**: `resetGame` → play → `loseLife` / `killBoss` → `endGame`; `togglePause` handles pause.
- **Accessibility**: a reduced-motion toggle (`reducedMotion`) gates screen shake, flashes and pulses; `announce()` writes to the `#srLive` ARIA live region.

## Conventions

- Keep the game in a single self-contained `index.html` unless the user asks to split it. Do not add external CDN scripts or fonts without asking.
- Match the existing style: `const`/`let`, 2-space indentation, single quotes in JS, `'use strict'` inside the IIFE, and no global variables.
- Do not change the logical resolution or the DOM ids the script depends on (`gameCanvas`, `overlay`, `startBtn`, `hud` elements, etc.) without updating every reference.
- When tuning gameplay, change values in the config tables rather than hard-coding new numbers in the update logic.
- Any new interactive control needs keyboard and touch support, and an `aria-label` or `title` where it is icon-only.

## Verifying changes

There are no automated tests. After changing game logic or UI:

1. Open `index.html` in a browser (or use the `run` skill / Playwright if available) and check the browser console for errors.
2. Play through start → stage clear → boss → game over, and confirm the HUD, pause, mute, and difficulty selector still work.
3. Check the mobile layout (≈375px wide) and touch controls when touching layout or input code.

## Git

- Development branch for this session: `claude/gifted-tesla-x7c00x`. Do not push to other branches without explicit permission.
- Only create pull requests when the user explicitly asks for one.
