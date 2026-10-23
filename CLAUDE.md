# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. There is no build step, package manager, test suite, or server. Everything lives in `index.html`:

- `<style>` (around lines 10–360): all CSS, including HUD, overlay, touch controls, and the theme.
- `<body>` (lines 361–421): DOM for the canvas, HUD, boss bar, top-right icon buttons, touch controls, and the start/pause overlay.
- `<script>` (lines 422 onward): one IIFE (`(function () { 'use strict'; ... })();`) containing the whole game.

`README.md` is a stub (title only).

## Running it

Open `index.html` directly in a browser, or serve the folder with any static server. No install step is needed.

## Code layout inside the script

Sections are marked with `// ---------- Name ----------` comments. Keep new code in the matching section.

- **Config tables**: `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes, cycled by stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`, with `minStage` gating), `BOSSES` (six boss skins, cycled by stage), `BOSS_PATTERNS` (boss bullet patterns).
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Access is unguarded, so wrap new reads/writes in try/catch if you add more.
- **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`) with a named `sfx` table. Audio starts only after a user gesture via `ensureAudio()`. Keep new sounds in `sfx`.
- **Adaptive quality**: `trackPerf` samples frame time over 45 frames and steps `quality` between 0 (full), 1 (medium), and 2 (low).
- **Input**: keyboard (arrows/WASD, Space fire, X/Shift bomb, P/Esc pause), mouse (move to steer, click to fire), and touch (left drag-to-fly joystick, FIRE and BOMB buttons).
- **Player and weapons**: `fireWeapon` handles two weapons, `vulcan` (spread shot) and `laser` (piercing lanes). Each has levels 1–3. `applyPowerUp` upgrades the level and then toggles the weapon after level 3.
- **Scoring**: `addScore` builds a combo `chain` (2.2 s window). The multiplier is `1 + min(9, floor(chain / 5))`. `breakChain` runs on a lost life or timeout.
- **Progression**: after `KILLS_PER_STAGE` kills, a boss spawns 1.2 s later (`spawnBoss`). Defeating it (`killBoss`) adds 500 points, increments `stage`, and resets the kill counter. Stage themes and boss skins rotate with `(stage - 1) % length`.
- **Power-ups**: `maybeSpawnPowerUp` drops `life` (6%), `bomb` (8%), or `weapon` (20%) from destroyed enemies.
- **Rendering**: `draw()` runs each frame. Helpers include `drawPlane`, `drawBossShip`, and `drawClouds`. Screen effects are `screenShake`, `flashHit`, and `flashBomb`. Reduced-motion mode disables shake, flash, and pulse.
- **Loop**: `requestAnimationFrame(loop)` drives `update(dt)` and `draw()`. `dt` is clamped to 0.05 s so a stalled tab doesn't cause a huge jump. Pause and game over are handled by `togglePause` and `endGame`.
- **Accessibility**: `announce()` writes to the `#srLive` aria-live region for important state changes. Keep that behavior when you add new messages.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file, no-build design is intentional.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow callbacks for event handlers, and short `//` section comments.
- Tuning values are inline numbers in the config tables and the update code. Change them in place. Don't introduce a settings framework.
- Do not add external dependencies, CDN scripts, or a build step without asking first.
- Preserve the mobile layout: the viewport meta disables zoom, and the `#touchControls` panel is shown only when `isTouchDevice` is true (the `.enabled` class). Check the layout at phone width when you change HUD or overlay CSS.

## Verification

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or serve the directory statically).
2. Play through at least one stage and one boss on each difficulty you touched.
3. Check the console for errors. Test touch controls in device emulation if you changed input or layout.
4. Confirm the best score, mute, difficulty, and reduced-motion settings still persist across reloads.

## Git

- Commit with clear messages that describe the gameplay or code change.
- Develop on the branch assigned to the session. Don't push to `main` directly.
