# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-page HTML5 canvas vertical shoot-'em-up in the style of Raiden. The whole game lives in one file, `index.html` (~1,670 lines): HTML, CSS in a `<style>` block, and JavaScript in a `<script>` block at the end of `<body>`. `README.md` is a placeholder.

There is no build step, package manager, test suite, linter, or CI configuration. Do not invent commands for these. To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Repository layout

```
index.html   # The entire game: markup (#container, #hud, #touchControls, #overlay), CSS, and JS
README.md    # Placeholder
```

## Working on the game

Everything is in one `<script>` block wrapped in an IIFE with `'use strict'`, using `const`/`let` and function declarations. Keep new code in that block and follow its conventions rather than adding modules or external dependencies.

Key sections, in the order they appear in the script:

- **DOM references and storage keys.** Element handles are fetched once at the top. `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
- **Tuning tables.** `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six designs, cycled by `(stage - 1) % BOSSES.length`). Balance changes usually belong here.
- **Audio.** Procedural Web Audio only (`beep`, `noiseBurst`, `sfx`). There are no audio files. Audio is created lazily through `ensureAudio()` on the first user gesture.
- **Input.** Keyboard (arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause), mouse on the canvas (steer and click to fire), and touch (`#moveZone` joystick, `#fireBtn`, `#bombBtn`).
- **Game state.** Module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, and so on). `resetGame()` reinitializes them.
- **Combat and pattern logic.** `spawnEnemy`, `spawnBoss`, `fireWeapon` (Vulcan and Laser, three power levels), `useBomb`, `update(dt)`, `updateBoss(dt)`, and the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **Flow.** `endGame`, `togglePause`, and `loop(now)`, which drives `update` and `draw` each frame.
- **Rendering.** `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Rendering is canvas 2D only.
- **Performance.** `trackPerf` samples frame times and lowers `quality` (0 full, 1 medium, 2 low). Particle counts scale with it.
- **Accessibility.** `announce()` writes to the `#srLive` live region. A reduced-motion toggle (`reducedMotion`) disables screen shake, flashes, and pulses, and it defaults to the `prefers-reduced-motion` media query.

When you change game logic, keep these in step:

- Stage progression: `stageKills` and `bossWarned` are reset in both `resetGame()` and the boss-defeat path.
- New power-up types need a spawn in `maybeSpawnPowerUp`, a pickup branch, and a color in `draw()`.
- New settings should persist through a `skyfighter.*` key. The existing `localStorage` reads and writes are not wrapped in `try`/`catch`, so guard new ones (storage can throw in private windows or when site data is blocked).

## Verification

Since there is no automated test setup, check changes by running the game:

1. Open `index.html` in a desktop browser and play through a stage transition and at least one boss fight.
2. Check the touch layout with browser device emulation, since the HUD and touch controls are responsive.
3. Toggle mute, pause, reduced motion, and difficulty, and confirm the values survive a page reload.
4. Confirm the browser console shows no errors.

If you use the `run` skill, it can launch the page for you.

## Conventions

- Match the surrounding code's style: two-space indentation, single quotes in JS, short comments that explain why, not what.
- Keep changes to `index.html` focused. The file is large, so use targeted edits rather than rewriting whole sections.
- Commit messages describe the change in plain language, in the style of the existing history (for example, `Add Sky Fighter: Raiden-style vertical shoot-em-up game`).
- Develop on the branch assigned to the task. Do not push to `main` directly.
