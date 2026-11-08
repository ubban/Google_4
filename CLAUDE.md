# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (about 1,670 lines: inline CSS, markup, and one IIFE-wrapped `<script>`). `README.md` is only a title stub.

There is no build system, package manager, bundler, framework, or test suite. Do not introduce one unless asked.

## Running it

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- No install step is needed.

## Code layout (`index.html`)

- **`<style>`** (approx. lines 10–360): HUD, overlay, touch controls, and the dark-mode `color-scheme`. Layout uses `#container` with a fixed-size `#gameCanvas`.
- **`<body>` markup** (approx. lines 362–420): the canvas, HUD (`#hud`), boss bar (`#bossBarWrap`), top-bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`), and the start/game-over overlay (`#overlay`, with difficulty buttons `.diffBtn`).
- **`<script>`** (approx. lines 422 onward), all inside one `(function () { 'use strict'; ... })()`:
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object. Audio is created lazily on the first user gesture.
  - Config constants: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six designs, cycled by stage).
  - Input: keyboard `keys` map, mouse on the canvas, on-screen joystick and buttons for touch.
  - Game state: `resetGame`, `update(dt)`, `draw()`, and `loop(now)` driving a `requestAnimationFrame` loop.
  - Combat: `fireWeapon`, `useBomb`, `spawnEnemy`, `spawnBoss`, the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, and so on), `updateBoss`, `killBoss`, `loseLife`, `endGame`.
  - Power-ups: `maybeSpawnPowerUp` (about 6% life, 8% bomb, 20% weapon) and `applyPowerUp`.
  - Persistence: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).

## Game rules worth knowing

- **Difficulty** (`DIFFICULTIES`): Easy (4 lives, 3 bombs, boss HP ×0.8), Normal (3 lives, 2 bombs, ×1.0), Hard (3 lives, 2 bombs, ×1.3). Spawn rate, enemy speed, and enemy fire chance also scale with difficulty. Changing difficulty is blocked mid-game.
- **Stages**: each stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss. The stage number selects the theme (`THEMES[(stage-1) % 6]`) and the boss design (`BOSSES[(stage-1) % 6]`).
- **Weapons**: Vulcan and Laser. A weapon power-up raises the level up to 3, then toggles to the other weapon at level 1.
- **Bombs** are a limited stock (`maxBombs`) refilled by bomb power-ups.
- **Pause** is `P` or `Esc`. Mute and reduced-motion toggles are in the top bar and persist in `localStorage`.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Preserve the single-file, no-dependency design.
- Keep the IIFE wrapper and `'use strict'`. Do not leak new globals.
- Use the existing naming style: `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constants, and `STORAGE_*` for `localStorage` keys (always prefixed `skyfighter.`).
- Gameplay numbers live in the constant tables near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer tuning them there rather than hard-coding values in logic.
- Play-field logical size is `BASE_W` × `BASE_H` (480 × 720). `resize()` scales the canvas to fit; keep draw code in these logical coordinates.
- Respect the reduced-motion setting when adding screen shake, flashes, or pulses (see `screenShake`, `flashHit`, `flashBomb`).
- Any new control must work on both keyboard/mouse and touch, and must update the `#controlsHint` text if it adds a key.
- Keep user-facing announcements going through `announce()` so screen readers get updates via `#srLive`.

## Verifying changes

There is no automated test suite or linter configured. To verify a change:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through: start on each difficulty, clear a stage to reach a boss, and confirm the stage banner and theme change.
3. Check pause, mute, reduced motion, and bomb/power-up HUD updates.
4. For layout or touch changes, check at phone width and with a touch emulation enabled.

## Git

- Development branches follow the session's instructions. Commit with clear messages and push with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.
