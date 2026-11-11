# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

| File        | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| `index.html`| The entire game: markup (`<body>`), CSS (`<style>`), and JS (`<script>`) in one file (~1,670 lines). |
| `README.md` | One-line project title. Keep it minimal.                                |

There is no build step, package manager, test suite, or linter configured. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

Playwright/Chromium can be used to load the page and take screenshots when verifying visual or gameplay changes.

## Architecture of `index.html`

The game script is one IIFE (`(function () { 'use strict'; ... })();`) so nothing leaks onto `window`. Keep new code inside that scope.

Major sections, in file order (search for the `// ----------` comment banners and function names):

1. **DOM references**: `const xxxEl = document.getElementById(...)` near the top of the script. IDs must match the markup.
2. **Persistence keys**: `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Stored in `localStorage`.
3. **Adaptive performance**: `trackPerf` lowers a `quality` level (0 full, 1 medium, 2 low) when the average frame time is too slow.
4. **Tuning data**: `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs).
5. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and `EXPLODE_SFX`. Audio is created lazily on the first user gesture.
6. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
7. **Layout**: `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the viewport.
8. **Input**: keyboard `keys` map, touch joystick (`joystickStart/Move/End`), mouse/pointer via `canvasPointFromEvent`, and touch buttons for fire and bomb.
9. **Game state**: `resetGame()`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore`/`breakChain` (kill-chain multiplier), `useBomb`, `fireWeapon`.
10. **Update and rendering**: `update(dt)` is the per-frame simulation; `draw()` renders it; `drawPlane`, `drawBossShip`, `drawClouds` are sprite helpers.
11. **Bosses**: `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring), `updateBoss`, `killBoss`.
12. **Flow**: `loseLife`, `endGame`, `togglePause`, and the `loop(now)` `requestAnimationFrame` driver at the bottom.

### Game-loop conventions

- `dt` is in seconds and is clamped to `0.05` in `loop` so a backgrounded tab cannot cause a huge jump.
- Simulation (`update`) and drawing (`draw`) run only when `running && !paused`.
- Frame-based counters (e.g. `frame++` in `update`) are used for patterns; time-based movement should multiply by `dt`.
- Entity collections are plain arrays that are filtered in place as entities die. Keep allocations per frame low, since the `quality` system exists to protect frame rate.

### Controls (keep the HUD hint in sync)

- Keyboard: Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mouse: move to steer, click to fire.
- Touch: drag in the left zone to fly, FIRE and BOMB buttons on the right.

The on-screen help text lives in `#controlsHint` (`index.html`, the overlay card). Update it whenever a control changes.

## Conventions

- **Single file**: Keep HTML, CSS, and JS together in `index.html` unless the user asks to split it. Do not introduce a bundler or framework.
- **No external dependencies**: Use the canvas 2D API and Web Audio directly. Do not add CDN scripts without asking.
- **Naming**: `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constant tables and keys (`DIFFICULTIES`, `BOSSES`, `STORAGE_*`). The code uses no classes.
- **Style**: 2-space indentation, single quotes in JS, semicolons on statements, arrow functions for callbacks, and `const` by default.
- **Accessibility**: Respect reduced motion. The `reducedMotion` flag (initialised from `prefers-reduced-motion` and the user's toggle) scales down screen shake in `screenShake()` and should guard any new flashing or shaking effect. Announce important state changes through `announce()`, which feeds the `#srLive` live region.
- **Responsive**: The layout must work on phones. Test touch changes in a touch-emulated viewport, not only with a mouse.
- **Persistence**: Wrap new `localStorage` reads and writes so the game still works if storage is unavailable, and use a `skyfighter.*` key name.

## Making changes

- Read the surrounding section before editing. Most systems are coupled through shared state (`enemies`, `bullets`, `score`, `lives`, `bombs`, and so on).
- When tuning balance, change the data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`) rather than hard-coding numbers in logic.
- When adding a boss pattern, add it to `BOSS_PATTERNS` and make sure `updateBoss` can select it.
- When adding a power-up, handle it in `applyPowerUp` and make sure `maybeSpawnPowerUp` can produce it.
- Verify gameplay changes by loading the page in a browser and checking the console for errors. There are no automated tests.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-rlf02p`). Do not push to `main` unless explicitly asked.
- Write commit messages that describe the gameplay or UI change, using the imperative mood (e.g. "Add ...", "Fix ...").
- Do not create pull requests unless the user explicitly asks for one.
