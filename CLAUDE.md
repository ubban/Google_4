# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up (Raiden-style) written in plain HTML, CSS and JavaScript. There is no build step, no package manager and no test suite.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (~lines 1–420 incl. `<style>`), inline `<script>` (~lines 422–1669). |
| `README.md` | Minimal repo readme. |

To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Code layout (`index.html`)

- **`<style>`** (starts ~line 10): all CSS, including HUD, overlay, touch controls and light/dark-friendly theming.
- **`<body>` markup** (~line 362 onward): `#container` holds the `<canvas id="gameCanvas">`, HUD (`#hud`, `#score`, `#best`, `#mult`, `#weaponName`, `#hearts`, `#bombs`), boss bar (`#bossBarWrap`), top-bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`, `#fireBtn`, `#bombBtn`, joystick), and the start/game-over `#overlay`.
- **`<script>`**: one IIFE wrapped in `'use strict'`. Everything is private to the closure; nothing is exposed on `window`.
  - **Config constants**: `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`. Balance changes usually belong here.
  - **Persistence**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
  - **Audio**: WebAudio helpers `ensureAudio`, `beep`, `noiseBurst`, `panFor`. Audio is created lazily on the first user gesture.
  - **Game state & logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore`/`breakChain` (combo multiplier), `update(dt)`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
  - **Boss bullet patterns**: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`.
  - **Rendering**: `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion`, `screenShake`, `flashHit`/`flashBomb`.
  - **Input**: mouse/pointer steering and firing (`canvasPointFromEvent`), keyboard (`window` `keydown` listener, ~line 769), and touch joystick (`joystickStart`/`joystickMove`/`joystickEnd`).
  - **Main loop**: `loop(now)` runs via `requestAnimationFrame`, clamps `dt` to 0.05 s, and only calls `update`/`draw` while `running && !paused`.
  - **Accessibility**: `announce()` writes to the `#srLive` live region; a reduced-motion toggle disables shake/flash/pulse effects.
  - **Performance**: `trackPerf(dt)` tracks frame timing.

## Controls (for reference when changing input code)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

## Conventions

- Keep the game in a **single self-contained HTML file** unless the user asks to split it. Do not add a bundler, framework or npm dependencies without explicit approval.
- Stay with vanilla JS and the existing IIFE structure. New globals should be `const`/`let` inside the IIFE.
- Use `const` by default, and keep the existing 2-space indentation and single quotes in the script.
- Game logic is frame-rate independent: scale movement and timers by `dt` (seconds), not by frame count.
- When adding user-visible events, call `announce()` so screen-reader users get the same information.
- Respect the reduced-motion setting for any new screen shake, flash or pulse effect.
- Persist new settings under the `skyfighter.` localStorage prefix, and wrap storage access so private-mode or disabled storage does not throw.

## Working in this repo

- Default branch work: develop on the branch assigned by the session (currently `claude/gifted-tesla-l73sxh`). Do not push to other branches without permission.
- There are no automated tests or linters. Verify changes by opening `index.html` in a browser (or a headless browser) and playing through the affected path: start, fire, bomb, pause, a stage transition, a boss fight and game over.
- Check both desktop and phone-width layouts when touching CSS or touch controls.
- Do not commit secrets or local config; the repo currently has none.

## Git

- Commit messages: short imperative summary line, followed by a body only when the reason is not obvious.
- Include the attribution trailers the session asks for at the end of commit messages.
- Do not open a pull request unless the user asks for one.
