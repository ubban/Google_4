# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines): inline CSS in `<style>`, markup in `<body>`, and all game logic in one `<script>` IIFE.

There is no build step, no bundler, no package manager, and no test suite. Nothing else is in the repo besides `README.md` (a placeholder), `index.html`, and this file.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Verify changes by playing them in a browser. There are no automated tests or linters to run.

## File layout (`index.html`)

| Lines (approx.) | Section |
|---|---|
| 1–360 | `<head>`, meta tags, and the inline `<style>` block (HUD pills, overlay card, touch controls, reduced-motion styles) |
| 361–421 | Body markup: `#container`, canvas `#gameCanvas`, HUD (`#hud`), boss bar, combo/stage banners, top buttons (mute, reduced motion, pause), touch controls, start overlay (`#overlay`) |
| 422–1669 | Single `<script>` IIFE with all game logic |

Key regions inside the script (search for the names):

- **Config constants**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `BASE_W`/`BASE_H` (480×720 logical resolution).
- **Persistence**: `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` map. Audio is synthesized with WebAudio; no audio files.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`.
- **Simulation**: `update(dt)`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
- **Input**: keyboard handlers, mouse steering (`mousemove` on the canvas, mapped via `canvasPointFromEvent`), a virtual joystick, and FIRE/BOMB touch buttons.
- **Main loop**: `loop(now)` drives `update` and `draw` via `requestAnimationFrame`. `dt` is clamped to 50 ms.

## Game rules and controls

- **Move**: Arrow keys or WASD, the on-screen joystick on touch devices, or mouse hover over the canvas (the plane glides toward the pointer). Keyboard/joystick input takes priority over mouse steering.
- **Fire**: Hold Space, or hold the FIRE button.
- **Bomb**: `X`, `Shift`, or the BOMB button. Bombs are limited per difficulty and clear enemy bullets and damage bosses.
- **Pause**: `P` or `Escape`, or the pause button.
- **Difficulty**: Easy / Normal / Hard (`DIFFICULTIES`). Sets spawn rate, enemy fire chance, lives, bombs, and boss HP multiplier.
- **Stages**: After `KILLS_PER_STAGE` (18) kills in a stage, a boss warning fires and the stage's boss spawns. Defeating the boss advances the stage. Backgrounds cycle through `THEMES`, and bosses cycle through `BOSSES` with patterns from `BOSS_PATTERNS`.
- **Weapons**: Vulcan and Laser, with 3 power levels, upgraded by power-ups.
- **Score**: Base values come from `ENEMY_TYPES[...].score` and boss scoring, multiplied by a kill-chain multiplier that `breakChain` resets.

## Conventions

- **Single file**: Keep the game in `index.html` unless the user asks for a split. Don't add a build system or dependencies without asking.
- **No external assets**: Graphics are drawn with canvas primitives and audio is synthesized. Don't add image, sound, or CDN dependencies without a reason.
- **Style**: Match the existing code: `'use strict'`, 2-space indentation, `const`/`let`, arrow functions for callbacks, and section headers written as `// ---------- Name ----------`.
- **Logical coordinates**: Game logic runs in the 480×720 logical space. `resize()` scales the canvas to the viewport, so don't use CSS pixels for gameplay positions.
- **Frame-rate independence**: Movement and timers use `dt` (seconds). Keep new timers and speeds in that unit.
- **Adaptive quality**: `trackPerf` lowers `quality` when frames run slow. Honor `quality` for expensive effects (particles, clouds) rather than adding unconditional work.
- **Accessibility**: Respect `reducedMotion` (screen shake, flashes, pulses) and keep the `#srLive` announcements for important state changes via `announce()`.
- **Persistence**: Wrap new `localStorage` use under a `skyfighter.*` key, and keep the existing pattern of reading with a fallback default.
- **Touch and desktop**: Any new control needs both a keyboard path and a touch path, and the touch controls only appear when `isTouchDevice` is true.

## Making changes

1. Check `git status` and work on the designated feature branch. Don't push to `main`.
2. Make the change in `index.html`, then open it in a browser (or a static server) and exercise the affected path: start, play, pause, bomb, boss, game over, and mobile layout if the change touches HUD or controls.
3. Commit with a descriptive message and push with `git push -u origin <branch>`.
4. Don't open a pull request unless the user asks for one.
