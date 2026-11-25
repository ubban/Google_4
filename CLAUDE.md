# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its main content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in a single HTML file on an HTML5
`<canvas>`.

## Repository layout

| Path        | Purpose                                                          |
|-------------|------------------------------------------------------------------|
| `index.html`| The whole game: inline CSS (`<style>`, ~lines 10–360), markup (`<body>`, ~lines 361–421), and one inline IIFE script (`<script>`, ~lines 422–1669). |
| `README.md` | Minimal project readme. Keep it short.                          |
| `CLAUDE.md` | This file.                                                       |

There is no build system, package manager, bundler, test suite, linter, or CI
config in the repository. Do not add one unless asked.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any
  static server (for example `python3 -m http.server`).
- No dependencies need to be installed.

## Architecture of `index.html`

All game code lives in one IIFE with `'use strict'`. Nothing is exported to
`window`. Keep it that way unless there is a clear reason to change it.

The script is organized in these sections, in order:

1. **DOM references**: `const xxxEl = document.getElementById(...)`, collected
   at the top.
2. **Constants**:
   - `STORAGE_*` keys (`skyfighter.*`, see below).
   - `DIFFICULTIES` (easy / normal / hard): spawn rate, enemy speed, enemy fire
     chance, lives, bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18). Stages cycle through `THEMES`.
   - `ENEMY_TYPES`: `grunt`, `interceptor`, `tank`, each with `minStage`.
   - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH,
     TITAN) with per-boss HP multipliers. Attack patterns are in `BOSS_PATTERNS`.
3. **Audio**: `ensureAudio()`, `beep()`, and `noiseBurst()`. Sound is synthesized
   with the Web Audio API. No audio files are loaded.
4. **Adaptive quality**: `trackPerf()` samples frame times and lowers the
   `quality` level (0 full, 1 medium, 2 low) when the frame rate drops.
5. **Layout and input**: `resize()` fits a `BASE_W × BASE_H` (480 × 720) logical
   canvas to the window. Keyboard state goes into `keys[e.code]`. Touch uses a
   joystick in `moveZone` plus `fireBtn` and `bombBtn`. Mouse steers the ship.
6. **Game state and logic**: `resetGame()`, `update(dt)`, `spawnEnemy()`,
   `spawnBoss()`, `fireWeapon()`, `useBomb()`, `applyPowerUp()`, `loseLife()`,
   `endGame()`, `togglePause()`.
7. **Rendering**: `draw()` and helpers such as `drawPlane()`, `drawBossShip()`,
   and `drawClouds()`.
8. **Main loop**: `loop(now)` clamps `dt` to 0.05 s, then calls `update` and
   `draw` while the game is running and not paused. It is driven by
   `requestAnimationFrame`.

Game timing uses `dt` (seconds). Keep new logic frame-rate independent.

## Controls (for reference when changing input code)

- Keyboard: Arrows or WASD to move, Space to fire, X or Shift for bomb, P or Esc
  to pause. Space's default is prevented so the page does not scroll.
- Mouse: move to steer, click to fire.
- Touch: drag in the left zone to fly, tap FIRE, tap BOMB.
- Input listeners are on `window`, except for the touch and mouse controls,
  which are on their own elements.

## Persistence

The game stores four keys in `localStorage`. Wrap any new storage access in
`try/catch`, because storage can be unavailable in private windows.

| Key                       | Value                              |
|---------------------------|------------------------------------|
| `skyfighter.bestScore`    | Highest score (number)             |
| `skyfighter.muted`        | `'1'` when muted                   |
| `skyfighter.difficulty`   | `easy`, `normal`, or `hard`        |
| `skyfighter.reducedMotion`| `'1'` or `'0'`. Defaults to the OS `prefers-reduced-motion` setting when unset. |

Existing key names are player-facing data. Do not rename them without a
migration.

## Accessibility

- Live status messages go through `announce()`, which writes to the `srLive`
  element.
- The `reduced-motion` body class (toggled by `updateMotionBtn()`) suppresses
  CSS animations and flashes such as the vignette, hit and bomb flashes, combo
  and stage banners, and the title icon (see the `body.reduced-motion` rules in
  `<style>`). `screenShake()` is not yet gated by this setting. New visual
  effects that flash or shake should respect it.
- Keep buttons as real `<button>` elements with accessible labels.

## Conventions

- Plain HTML, CSS, and ES2015+ JavaScript. Do not introduce frameworks or CDN
  dependencies without a clear request.
- Keep the single-file structure. If `index.html` grows substantially, discuss
  splitting it into separate CSS and JS files before doing so.
- Match the existing style: 2-space indentation, single quotes in JavaScript,
  `const` by default, section comments in the form
  `// ---------- Name ----------`.
- Prefer small, targeted edits over rewriting large sections of the file.

## Verifying changes

Nothing is automated, so verify by hand:

1. Load `index.html` in a browser and check the browser console for errors.
2. Start a game, play through at least one enemy wave, and confirm that firing,
   bombs, pause, and game over all work.
3. For UI or layout changes, check both desktop and narrow (phone-width) viewports.
4. If you have a headless browser available, use it for scripted checks.

## Git workflow

- Develop on the branch assigned for the session (currently
  `claude/gifted-tesla-b651tx`). Do not push to other branches without
  explicit permission.
- Do not open a pull request unless the user asks for one.
- Keep commit messages focused on the change. Do not include model identifiers
  in commits, PR titles, or PR bodies.
