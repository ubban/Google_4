# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up written as a single-page HTML5 canvas app.

| File         | Purpose                                                        |
|--------------|----------------------------------------------------------------|
| `index.html` | The entire game: inline CSS, markup, and one IIFE-wrapped `<script>`. |
| `README.md`  | Placeholder title only.                                        |

There is no build system, package manager, test suite, or linter configured.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server
  (for example `python3 -m http.server`) and visit it.
- No install step is needed.

## Architecture of `index.html`

The file is organized top to bottom as:

1. **`<style>`**: layout, HUD pills, glassmorphic overlay card, touch controls, reduced-motion
   rules, and screen-reader-only helpers (`.sr-only`).
2. **Markup**: `#container` holds the `<canvas id="gameCanvas">`, HUD (`#hud`), boss bar,
   combo/stage banners, top bar buttons (mute, reduced motion, pause), touch controls, and the
   start/pause/game-over `#overlay`.
3. **Script**: a single `(function () { 'use strict'; ... })();` IIFE. Nothing is exposed on
   `window`. Keep new code inside this IIFE.

Key sections inside the script, in order:

- **DOM references**: cached once at the top with `getElementById` / `querySelectorAll`.
- **Config tables**: `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss
  HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
- **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`,
  `difficulty`, `reducedMotion`). Every read and write should tolerate storage being unavailable.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst`
  helpers. Audio starts only after a user gesture.
- **Input**: keyboard, mouse, and touch. Touch uses a left-side joystick plus FIRE and BOMB
  buttons. Touch handlers call `preventDefault` with `{ passive: false }`.
- **Game state and update**: `resetGame`, `update(dt)`, spawning (`spawnEnemy`, `spawnBoss`,
  `maybeSpawnPowerUp`), weapons (`fireWeapon`, weapon levels), bombs (`useBomb`), scoring with a
  kill-chain multiplier (`addScore`, `breakChain`), and lives (`loseLife`, `endGame`).
- **Boss**: `updateBoss` drives attack patterns from `BOSS_PATTERNS`; `killBoss` advances stages.
- **Rendering**: `draw()` with helpers `drawPlane`, `drawBossShip`, `drawClouds`, and
  `spawnExplosion`. Canvas size is set in `resize()` against a `480x720` base resolution.
- **Main loop**: `loop(now)` clamps `dt` to 0.05s, updates and draws only when `running && !paused`,
  and calls `trackPerf`.

## Conventions

- **Single file.** Keep CSS, markup, and JS in `index.html` unless the user asks for a split.
  Do not introduce a bundler, framework, or npm dependencies without an explicit request.
- **Vanilla JS only**, ES2015+ syntax already used in the file (`const`/`let`, arrow functions,
  template literals). Match the existing style: 2-space indentation, single quotes, semicolons.
- **Respect reduced motion.** Screen shake, hit flashes, and the low-health pulse must be
  gated by the `reducedMotion` flag, which also honors `prefers-reduced-motion`.
- **Accessibility.** Announce important state changes through the `#srLive` live region
  (`announce()`), and keep buttons labelled (`title` / visible text).
- **Mobile first.** Changes must keep working with touch controls and the `touchControls` layout.
- **Balance changes** belong in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`,
  `BOSS_PATTERNS`) rather than being hard-coded in logic.

## Testing and verification

There is no automated test suite. For gameplay or rendering changes:

- Load the page in a browser and play through at least one stage, a boss fight, and a game over.
- Check the three difficulty levels, mute and reduced-motion toggles, and pause/resume.
- Check touch layout at phone width, with no horizontal page scroll.
- Confirm that a best score persists across reloads.

If a browser cannot be run in the environment, say so explicitly and verify by reading the code.

## Git workflow

- Develop on the branch the task designates. Do not push to other branches without permission.
- Use clear, descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless the user asks for one.
