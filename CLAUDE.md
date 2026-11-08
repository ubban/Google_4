# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only app is **Sky Fighter**, a
Raiden-style vertical shoot-em-up written as a single self-contained HTML5
canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: inline `<style>`, markup, and one `<script>`. |
| `README.md` | Minimal project title only.                                    |

There is no build system, package manager, test suite, or bundler. Nothing
needs to be installed. To run the game, open `index.html` in a browser (or
serve the directory with any static file server).

## Layout of `index.html`

- **Lines ~1–360**: `<head>` metadata (the `<meta name="description">` is the
  game pitch) and the `<style>` block. HUD, overlay card, touch controls,
  stage banner, and vignette/flash effects are styled here. Reduced-motion
  rules use the `body.reduced-motion` class.
- **Lines ~361–421**: `<body>` markup: the `#gameCanvas` element, HUD elements,
  overlay, difficulty buttons, touch controls, and pause/mute/motion buttons.
- **Lines ~422–end**: one IIFE (`'use strict'`) containing all game logic.
  Key sections, in file order:
  - Constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`,
    `BOSSES`, `BOSS_PATTERNS`, and `STORAGE_*` keys.
  - Audio: `ensureAudio`, `beep`, `noiseBurst`. Sound is synthesized with the
    Web Audio API; there are no audio asset files.
  - Input: keyboard handlers, pointer/joystick helpers, and touch listeners.
  - State and spawning: `resetGame`, `spawnEnemy`, `spawnBoss`,
    `maybeSpawnPowerUp`, `applyPowerUp`.
  - Combat: `fireWeapon`, `useBomb`, `updateBoss`, boss attack functions
    (`fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`, etc.), `killBoss`,
    `loseLife`, `endGame`.
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`.
  - Main loop: `update(dt)` runs game logic, `draw()` renders, and `loop(now)`
    drives both through `requestAnimationFrame`. Each frame's `dt` is clamped
    to 0.05 s.

## Gameplay and design constants

- **Controls**: arrows or WASD to move; Space or hold fire for the weapon;
  X, Shift, or the bomb button to use a smart bomb; P or Escape to pause.
  Touch devices get a joystick, fire button, and bomb button.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal`, `hard`. Each sets spawn
  rates, enemy speed, shoot chance, starting lives, starting bombs, and a boss
  HP multiplier.
- **Stages**: each stage ends after `KILLS_PER_STAGE` (18) kills and then
  spawns a boss from `BOSSES`. Background `THEMES` rotate by stage.
- **Persistence** (`localStorage`, all keys prefixed `skyfighter.`):
  `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every read and write
  is a direct `localStorage` call, so keep them working in browsers where
  storage is unavailable or throws.
- **Accessibility**: a `prefers-reduced-motion` media query sets the initial
  reduced-motion flag. The user can override it with the motion toggle. Live
  announcements go through the `#srLive` region via `announce()`.

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Do not add
  a bundler, framework, or package manager without asking.
- Use plain ES2017+ JavaScript that runs in current evergreen browsers. Match
  the existing style: 2-space indent, single quotes, `const`/`let`, arrow
  functions for callbacks, and no semicolon-free code.
- Game tuning values are inline constants. Change them in the existing
  constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) rather than
  scattering magic numbers.
- Keep the game loop frame-rate independent: scale movement and timers by `dt`.
- Preserve the `reduced-motion` and touch-control paths when changing visuals
  or input.

## Verifying changes

There are no automated tests or linters. To check a change:

1. Open `index.html` in a browser, or serve it with `python3 -m http.server`.
2. Confirm the page loads without console errors.
3. Play through the start, a stage transition, a boss fight, bomb use, pause,
   and game over. Confirm the best score persists after a reload.
4. For UI or touch changes, test a narrow viewport or touch emulation.

Headless Chromium with Playwright can automate step 2 and 3 if a scripted
check is useful.

## Git workflow

- Default branch: `main`. Feature work for this session goes on
  `claude/gifted-tesla-oip9l1`.
- Write clear commit messages that describe the change, with a short body for
  non-trivial work.
- Do not open a pull request unless the user asks for one.
