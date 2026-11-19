# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file, `index.html`, with inline CSS and JavaScript and no external dependencies.

Other files:
- `README.md`: a placeholder title only.

There is no package manager, build step, test suite, or linter. Do not add one unless asked.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` and visit `http://localhost:8000`).
- Verify changes by playing them in a browser. Syntax-checking alone is not enough for gameplay changes.

## Code layout (`index.html`)

- **Lines ~1–362**: inline `<style>`. Layout is a full-viewport flex container with a centered `<canvas id="gameCanvas">`. The HUD, overlays, vignette, and touch controls are absolutely positioned on top. Tap-safe, phone-width layout is intended.
- **Lines ~363–421**: markup for the canvas, HUD, overlay (title/start/game-over), touch joystick and fire/bomb buttons, and the stage banner.
- **Lines ~422–1669**: one IIFE (`'use strict'`) containing all game logic. Major sections, in order:
  - Config constants: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (six stage backgrounds), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled per stage), and `BOSS_PATTERNS` (bullet patterns).
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and per-enemy explosion SFX, built on the Web Audio API.
  - UI helpers: HUD rendering (`renderHearts`, `renderBombs`), flashes, combo pops, stage banners, screen shake, and difficulty/mute/motion toggles.
  - Input: keyboard handlers, pointer and touch joystick handling, and `canvasPointFromEvent`.
  - Simulation: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, and `endGame`.
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
  - Main loop: `loop(now)` uses `requestAnimationFrame`, caps `dt` at 0.05 s, and skips `update`/`draw` while paused or not running.

Coordinates use a fixed logical canvas of `BASE_W = 480` by `BASE_H = 720`. `resize()` scales it to fit the window.

## Controls

- **Move**: arrow keys or WASD, or the on-screen joystick on touch devices.
- **Fire**: hold Space, or the fire button.
- **Bomb**: X or Shift, or the bomb button. Bomb count depends on difficulty.
- **Pause**: P or Escape.
- **Mute** and **reduced motion** are toggled from the HUD buttons.

## Persistence

The game stores settings and high score in `localStorage` under these keys. Keep them stable. Changing a key silently resets player data.

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads and writes are not wrapped in try/catch. If you add new storage, handle the case where storage is unavailable (private windows, blocked site data).

## Conventions

- Keep everything in `index.html` unless the user asks for a split. New constants go near the existing config block, and new enemy, boss, or theme data follows the existing object shapes.
- Preserve the `'use strict'` IIFE and do not add globals.
- Keep the accessibility hooks intact: `srLive` announcements via `announce()`, and the reduced-motion path that disables screen shake and flashes.
- Respect `prefers-reduced-motion` for any new visual effect that moves or flashes.
- Preserve the touch layout. The page sets `touch-action: none` and `user-select: none` on purpose, so do not remove them.
- Match the existing style: two-space indentation, single quotes in JavaScript, and short comments only where the intent is not obvious.

## Git workflow

- Develop on the branch named in the session instructions (currently `claude/gifted-tesla-y083gs`). Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Commit messages should describe the gameplay or code change in the imperative mood.
