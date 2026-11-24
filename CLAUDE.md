# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. The whole game lives in a single file, `index.html` (~1,670 lines): inline `<style>`, the markup (including a `<canvas id="gameCanvas">`), and one inline `<script>` wrapped in an IIFE with `'use strict'`.

Files:

- `index.html` — the entire game (HTML, CSS, JS).
- `README.md` — placeholder; currently only a title line.
- `CLAUDE.md` — this file.

There is no `package.json`, build step, bundler, test suite, linter, or CI config. Do not invent one. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Architecture of `index.html`

The inline script is organized in sections, roughly in this order:

1. **DOM lookups** — every element is fetched once up front by `id` (`gameCanvas`, `overlay`, `startBtn`, `hearts`, `bombs`, `touchControls`, `moveZone`, `fireBtn`, `bombBtn`, `pauseBtn`, `muteBtn`, `motionBtn`, `bossBarWrap`, etc.). Keep this pattern: new UI elements should be added to the HTML and looked up here.
2. **Accessibility helpers** — `announce(text)` writes to the `srLive` live region for screen readers.
3. **Adaptive quality** — `trackPerf(dt)` keeps a rolling window of frame times and steps `quality` (0 full, 1 medium, 2 low) up or down to hold roughly 40–56 FPS.
4. **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes; cycled by stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six bosses, with colors and `hpMul`). Balance changes should go here first.
5. **Persistence** — `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write is a direct `localStorage` call; keep new keys under the same prefix.
6. **Audio** — a lazily created `AudioContext` (`ensureAudio()`), with `beep`, `noiseBurst`, and an `sfx` object of named sounds. Audio must only start after a user gesture; call `ensureAudio()` from input handlers.
7. **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
8. **Input** — keyboard (`keys[e.code]`), a virtual joystick and fire/bomb buttons for touch (`touchstart`/`touchmove`/`touchend`), mouse and pointer-position tracking on the canvas. The canvas scales to fit via `resize()` using `BASE_W = 480`, `BASE_H = 720`.
9. **Game state and entities** — `resetGame()`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), collision (`rectsOverlap`), particles (`spawnExplosion`), scoring and combos (`addScore`, `breakChain`), weapons (`fireWeapon`, `applyPowerUp`), bombs (`useBomb`), and lives (`loseLife`, `endGame`, `togglePause`).
10. **Boss patterns** — `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`); `updateBoss` and `killBoss` drive them.
11. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()` (all Canvas 2D; no WebGL).
12. **Main loop** — `loop(now)` computes a `dt` clamped to 50 ms, calls `trackPerf`, then `update(dt)` and `draw()` while `running && !paused`, and schedules itself with `requestAnimationFrame`. The game starts with `requestAnimationFrame(loop)`; the Start button calls `resetGame()`.

Game flow: overlay (title/start) → play. Each stage ends with a boss once `KILLS_PER_STAGE` kills are reached; defeating the boss (`killBoss`) enters the next stage, which uses the next theme in `THEMES` (cycled with modulo). Losing all hearts calls `endGame()` (updates best score, shows stats). Pause toggles via `togglePause()`.

## Conventions

- **Keep it a single file.** Do not split the game into modules, add a bundler, or add npm dependencies unless the user explicitly asks. Everything (CSS, JS, assets) stays inline and the game should keep working offline with no external requests.
- **Dependency-free vanilla JS.** Use ES2015+ features already in use (`const`/`let`, arrow functions, template strings). Do not introduce frameworks or libraries.
- **Respect the IIFE and `'use strict'`.** Do not leak globals; new helpers go inside the IIFE.
- **Name constants in `UPPER_SNAKE_CASE`** and tuning values in the data tables above rather than hard-coding magic numbers in update/draw code.
- **Respect the reduced-motion and mute settings.** Any new visual effect that shakes, flashes, or pulses should check `reducedMotion`; any new sound should go through the existing `sfx`/`beep`/`noiseBurst` helpers so `muted` is honored.
- **Keep touch and keyboard working.** Any new player action needs a keyboard binding and a touch control (or an on-screen equivalent).
- **Performance matters.** The main loop runs every frame. Avoid per-frame allocations where practical, and respect `quality` when adding particles or expensive effects.
- **Keep the HUD accessible.** Update text through the existing elements and use `announce()` for important state changes.

## Working in this repo

- Develop on the branch the session was started on (currently `claude/gifted-tesla-ve5sb1`). Do not push to other branches without permission.
- There are no automated tests. Verify changes by loading `index.html` in a browser and playing through the affected path: start, fire, bomb, pause/resume, boss fight, game over, and the difficulty and mute toggles. For visual or input changes, check both desktop (keyboard/mouse) and a touch-sized viewport.
- When you change balance values (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`), say so in the commit message so the tuning history is easy to follow.
- Keep the `skyfighter.*` `localStorage` key names stable. Renaming them silently resets players' best scores and settings.
- Do not commit generated or local-only files. The repo currently contains only `index.html`, `README.md`, and `CLAUDE.md`.

## Open items / known gaps

- `README.md` is a placeholder. Expand it with a short description, how to run the game, and controls if the user wants that.
- No tests, linting, or formatting config exist yet.
