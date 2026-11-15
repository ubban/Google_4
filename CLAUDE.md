# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single static page. There is no build step, no package manager, and no test suite.

- `index.html` (~1,670 lines): the entire game. All CSS is in one `<style>` block (starts ~line 10), all markup is in the body (canvas starts ~line 363), and all JavaScript is in one IIFE inside a `<script>` block (starts ~line 422).
- `README.md`: placeholder. It only has the repo name and a timestamp.

Git: default branch is `main`. Development for this task happens on `claude/gifted-tesla-4cu9rd`.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`).
- There are no tests, linters, or build commands. Verify changes by playing the game in a browser. Check at least the start screen, a full game loop, pause/resume, game over, and the touch controls if the change touches input.

## Code structure (inside `index.html`)

The script is organized into commented sections, roughly in this order:

1. **DOM refs and constants**: element lookups, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses with palettes and HP multipliers).
2. **Adaptive performance** (`trackPerf`): averages the last 45 frame times and steps a `quality` level (0 full, 1 medium, 2 low) up or down.
3. **Settings and persistence**: best score, mute, difficulty, and reduced-motion preference.
4. **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `sfx.*`): Web Audio synthesis only. No audio files. Audio is created lazily on the first user gesture.
5. **HUD and effects**: hearts, bombs, combo pop, stage banner, screen shake, flash, and screen-reader announcements (`announce`).
6. **Input**: keyboard (arrows/WASD to move, Space to fire, X or Shift for bomb, P or Escape to pause), on-screen joystick and buttons for touch, and mouse on the canvas.
7. **Game state and logic**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, power-ups (`applyPowerUp`), weapons (`fireWeapon`), bombs (`useBomb`), score and multiplier chain (`addScore`, `breakChain`), `loseLife`, `endGame`, `togglePause`.
8. **Boss patterns**: `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` and `killBoss` drive them.
9. **Rendering**: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`. Drawn with Canvas 2D at a fixed logical size of 480×720 (`BASE_W`, `BASE_H`), scaled by `resize()`.
10. **Main loop**: `loop(now)` clamps `dt` to 50 ms and only calls `update` and `draw` while running and not paused. It starts with `requestAnimationFrame`.

Everything lives in a single IIFE, so nothing is exposed on `window`. Keep new code inside that scope unless there is a reason not to.

## Persistence

The game uses `localStorage` with these keys. Keep the prefix `skyfighter.` for any new key:

- `skyfighter.bestScore`
- `skyfighter.muted` (`'1'` or `'0'`)
- `skyfighter.difficulty` (`easy`, `normal`, or `hard`)
- `skyfighter.reducedMotion` (`'1'` or `'0'`; if unset, follows `prefers-reduced-motion`)

Reads and writes are not wrapped in try/catch today. If you add storage that must work in private browsing or with blocked site data, wrap it.

## Conventions

- Plain HTML, CSS, and vanilla JavaScript (ES2015+ syntax already in use: `const`/`let`, arrow functions, template literals). Do not introduce a framework, bundler, or npm dependency without an explicit request.
- Keep the single-file structure. Do not split out modules or assets unless asked.
- Follow the existing style: 2-space indent, single quotes in JS, section headers as `// ---------- Name ----------`.
- Game balance lives in the constant tables near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer tuning those over changing logic.
- Keep accessibility hooks working: `srLive` announcements, the `reduced-motion` body class, and keyboard controls.
- Keep the layout usable on touch devices. The touch controls get the `enabled` class when `ontouchstart` exists or `navigator.maxTouchPoints > 0` (~line 778).

## Working in this repo

- Make small, focused changes, and say in your summary what you verified in the browser.
- Do not commit or push unless asked. When you do commit, use a clear message that describes the change.
- Do not create a pull request unless asked.
