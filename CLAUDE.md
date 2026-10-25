# CLAUDE.md

Guidance for working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs entirely in the browser. The whole game is one file, `index.html`, with inline CSS and one inline `<script>` block. `README.md` is a placeholder.

There is no build step, package manager, bundler, framework, or test suite. Do not add one unless asked.

## Running the game

Open `index.html` in a browser. No server is required.

For a quick local check, serve the directory with any static server, for example `python3 -m http.server`, and load it at `localhost`.

## Layout of `index.html`

- **`<style>` (around lines 10–359):** all styling. Includes the `.reduced-motion` body class, which tones down shake, flash, and pulse effects.
- **`<body>` markup (around lines 361–420):** a `#container` holding the `<canvas id="gameCanvas">`, the HUD (`#hud`, `#bossBarWrap`, `#topBar`), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **Script (around lines 422–1667):** a single IIFE with `'use strict'`. Sections, roughly in file order:
  - DOM references and storage keys (`STORAGE_*`).
  - Config tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is created lazily on the first user gesture.
  - HUD helpers and effects: `renderHearts`, `renderBombs`, `flashHit`, `screenShake`, `announce` (screen-reader live region).
  - Input: mouse, keyboard, and touch joystick handling (`joystickStart`/`joystickMove`/`joystickEnd`, `canvasPointFromEvent`).
  - Game state: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`.
  - Weapons: `fireWeapon` (vulcan and laser, each with levels 1–3) and bullet patterns.
  - Boss patterns: the `BOSS_PATTERNS` array and the `fire*` helpers. `updateBoss` and `killBoss` drive them.
  - Flow: `update(dt)`, `loseLife`, `endGame`, `togglePause`.
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
  - Main loop: `loop(now)`, which uses `requestAnimationFrame` and caps `dt` at 0.05 s.

## Game rules worth knowing

- **Difficulties** (`DIFFICULTIES`): easy, normal, hard. Each sets spawn rates, enemy speed, shoot chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages:** the stage advances every `KILLS_PER_STAGE` (18) kills. Each stage picks a theme from `THEMES`, cycling through the list. Stage 2 and later can spawn tanks, and each stage has a boss from `BOSSES`.
- **Power-ups** (`maybeSpawnPowerUp`): life (6%), bomb (8%), weapon (20%). Weapon pickups level up the current weapon to level 3, then switch between vulcan and laser.
- **Adaptive quality** (`trackPerf`): the game tracks frame time over 45 frames and lowers or raises a `quality` level. Keep new effects cheap, or gate them on `quality`.
- **Persistence:** `localStorage` keys are `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Reads must tolerate missing or invalid values: a missing key returns `null`, so fall back to the defaults the code already uses.
- **Reduced motion:** defaults to the `prefers-reduced-motion` media query and can be toggled. Any new screen shake, flash, or pulse effect must respect `reducedMotion`.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Match the existing style: 2-space indent, single quotes, `const` and `let`, arrow functions for callbacks, and short section comments where they help.
- Keep new config values in the existing config tables rather than hard-coding them in logic.
- Keep the game loop's `update` and `draw` separate. Logic goes in `update`, and only drawing goes in `draw`.
- Use `dt` (seconds) for all motion so behavior does not depend on frame rate.
- Any user-visible state change (score, lives, stage, weapon, pause) should be reflected in the DOM HUD and, where useful, announced through `announce`.
- Keep the layout working at phone width. Touch controls are shown only on touch input, so check both paths when changing input.
- Do not add external scripts, fonts, or CDN links without a clear reason. The page is self-contained today.

## Verification

There are no automated tests or linters. Verify changes by:

1. Opening `index.html` in a browser and playing through at least one stage transition and one boss fight.
2. Checking all three difficulties, pause and resume, mute, and reduced-motion toggles.
3. Checking the touch layout at a phone-sized viewport.
4. Checking the browser console for errors.

If you add a test harness, document how to run it here.

## Git

- Development branch for this work: `claude/gifted-tesla-iatazh`.
- Remote: `origin` → `https://github.com/ubban/Google_4`.
- Keep commits focused and write clear messages that say what changed and why.
