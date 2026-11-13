# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** is a single-file, browser-based vertical shoot-'em-up (Raiden-style), written in vanilla HTML, CSS and JavaScript. There is no build step, no bundler, no package manager and no test suite.

Files:

- `index.html` — the entire game: markup (~lines 1–421), CSS (inline `<style>`, ~lines 10–361), and one IIFE-wrapped `<script>` (starts ~line 422) containing all game logic, rendering and audio.
- `README.md` — minimal project readme.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). Nothing needs installing.

## Architecture (inside the `<script>`)

- **Rendering:** a single `<canvas id="gameCanvas">` drawn with the 2D context. Logical resolution is `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()`.
- **Loop:** `requestAnimationFrame(loop)`. `dt` is clamped to 0.05s. Each frame runs `update(dt)` then `draw()`, only while `running && !paused`.
- **Adaptive quality:** `trackPerf()` moves a `quality` level (0 full, 1 medium, 2 low) based on a rolling average of frame times. Effects code should respect this level.
- **Tunables (top of script):** `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Prefer changing these data tables over hard-coding values in logic.
- **Game state:** module-level `let` variables reset by `resetGame()`. Entity arrays (enemies, bullets, power-ups, explosions) are updated in `update()` and drawn in `draw()`.
- **Bosses:** `spawnBoss()`, `updateBoss(dt)`, `killBoss()`, with attack patterns as functions (`fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`, etc.) selected from `BOSS_PATTERNS`.
- **Audio:** Web Audio API, created lazily in `ensureAudio()` (must run from a user gesture). Helpers: `beep()`, `noiseBurst()`, `panFor()`.
- **Input:** keyboard (arrows/WASD, Space fire, X/Shift bomb, P/Esc pause), mouse (`canvasPointFromEvent`), and touch (virtual joystick + FIRE/BOMB buttons). Keep all three working when changing controls.
- **UI:** HUD elements and overlays are plain DOM nodes updated from script (`scoreEl`, `renderHearts()`, `renderBombs()`, `showStageBanner()`, etc.). Screen-reader text goes through `announce()` into the `#srLive` region.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads and writes are direct and unguarded. If you add new keys, follow the same prefix convention.

## Accessibility and motion

- A reduced-motion mode (`reducedMotion`, toggled by `#motionBtn`) disables screen shake, screen flash and pulse effects. It defaults from `prefers-reduced-motion`. Any new visual effect that shakes, flashes or pulses must honor this flag.
- Live announcements go through `announce()`. Buttons carry `title` attributes.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. The single-file design is intentional.
- Stay with vanilla JS and the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, and the existing naming (`spawnX`, `fireX`, `drawX`, `renderX`).
- Do not add external dependencies or CDN scripts without a specific request.
- Keep the code inside the IIFE and `'use strict'`. Do not add new globals.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage, a boss fight, a power-up pickup, a bomb, and a game over. Confirm the best score persists after reload.
3. Check all three difficulties, mute, and reduced-motion toggles.
4. For control or layout changes, test keyboard, mouse and a touch-sized viewport.

Headless browser checks (Playwright) are fine for smoke tests, but they do not replace playing the game.

## Git

- Development branch: `claude/gifted-tesla-pxm96q`. Do not push to other branches without explicit permission.
- Keep commits focused. Only commit or push when asked.
