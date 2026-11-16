# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one ~1,670-line file. |
| `README.md` | One-line project title. |

There is no build step, no package manager (`package.json` does not exist), no test suite, no linter config, and no external scripts or stylesheets. The only network-free dependencies are browser APIs (Canvas 2D, Web Audio, `localStorage`, `matchMedia`).

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

There are no install or build commands to run.

## Code layout inside `index.html`

The script is one `'use strict'` IIFE, so all state is `const`/`let` inside it. Rough order within the script:

1. **DOM references**: `const` lookups by element id (`scoreEl`, `bossBarFill`, `touchControls`, ...). If you add a UI element, add its lookup here.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`. Persisted values use the `skyfighter.*` prefix.
3. **Tuning tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, enemy fire chance, starting `lives`, starting `bombs`, `bossHpMul`.
   - `KILLS_PER_STAGE` (currently 18): kills needed to advance a stage.
   - `THEMES`: background/cloud palettes per stage.
   - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`): size, HP, speed multiplier, color, score, and `minStage` gating.
   - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with colors and an `hpMul`.
4. **Audio**: `ensureAudio()` lazily creates the `AudioContext`. `beep()` and `noiseBurst()` are primitives; `sfx` holds named sounds (`bomb`, `bossHit`, `bossTelegraph`, `bossDown`, `stageClear`, `powerup`, ...).
5. **UI helpers**: `renderHearts`, `renderBombs`, `showStageBanner`, `showComboPop`, `flashHit`, `flashBomb`, `screenShake`, `announce` (screen-reader live region).
6. **Layout**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the viewport, capped at 1.6x.
7. **Input**: keyboard (`keys` map; `X`/`Shift` bomb), mouse steering and click-to-fire, and touch (`joystickStart`/`joystickMove`/`joystickEnd`, fire and bomb buttons).
8. **Game state**: `resetGame()` initializes `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, and `boss`.
9. **Simulation**: `spawnEnemy()`, `update(dt)`, `updateBoss(dt)`.
10. **Rendering**: `draw()` and helpers `drawPlane`, `drawBossShip`, `drawClouds`.
11. **Main loop**: `loop(now)` drives `update` and `draw` via `requestAnimationFrame`. `trackPerf` samples frame times.

Power-up types spawned in `update` are `life`, `bomb`, and `weapon` (weapons are `vulcan` and `laser`).

## Conventions

- **Keep it one file** unless the user asks to split it. Everything (CSS, JS, assets as `data:` URIs) lives in `index.html`.
- **No external dependencies.** Don't add CDN `<script>` or `<link>` tags, or `npm` packages, without asking. The page works offline as-is.
- **Tune through the constant tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, ...) rather than hard-coding numbers in the update loop.
- **Respect reduced motion.** `body.reduced-motion` disables stage-banner pulses and bomb flashes in CSS; new visual effects should honor the same toggle (`reducedMotion`).
- **Accessibility:** announce important state changes through `announce()` so screen readers get them. Keep tap targets usable on touch devices.
- **Mobile first:** the layout must work at phone widths. The page uses `touch-action: none` and `user-select: none`; don't remove those without a reason.
- **Persist only through the `skyfighter.*` keys** in `localStorage`, and wrap access in try/catch if you add new keys, since storage can be unavailable in private windows.

## Working on the game

- Time-based updates use a `dt` passed into `update`/`updateBoss`. Keep new movement frame-rate independent.
- When changing difficulty or balance, adjust the tables first and verify all three difficulties.
- After any change, play-test in a browser at both desktop and phone viewport sizes: start, fire, bomb, collect a power-up, clear a stage, fight a boss, and reach game over. Check the console for errors.
- Only commit when asked. Develop on the branch the session specifies.

## Git

- Commit messages: short imperative summary line, optional body explaining why.
- Don't force-push shared branches.
