# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static project containing **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–359), and JavaScript (`<script>`, ~lines 422–1667). |
| `README.md` | Minimal placeholder. |
| `CLAUDE.md` | This file. |

There is no build step, no package manager, no bundler, and no framework. Do not add dependencies or tooling unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server`. No install step is required.

## Architecture of `index.html`

The script is one IIFE with `'use strict'`. Its sections, in order:

1. **DOM lookups**: `getElementById` handles for the HUD, overlay, touch controls, and buttons. IDs are referenced directly, so renaming an element in the markup means updating the matching lookup.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`, all in `localStorage`. Changing a key name silently resets that user setting.
3. **Adaptive performance**: `trackPerf` samples frame times and moves `quality` between 0 (full), 1 (medium), and 2 (low).
4. **Tuning tables**: `DIFFICULTIES` (easy / normal / hard), `THEMES` (background palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses with HP multipliers), and `KILLS_PER_STAGE`. Most balance changes belong here rather than in the update logic.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio`) with synthesized sound effects (`beep`, `noiseBurst`, the `sfx` helpers). Audio is created only after a user gesture.
6. **HUD and effects**: hearts and bombs rendering, combo pops, stage banners, screen shake, and hit flash.
7. **Input**: keyboard (`keys` map), mouse (steer and fire on the canvas), and touch (virtual joystick, FIRE, and BOMB buttons). Touch controls are shown only when `isTouchDevice` is true.
8. **Game state**: `resetGame()` initializes the state variables, and `update(dt)` runs one simulation step.
9. **Weapons and patterns**: `fireWeapon`, power-up application, and the `fire*` helpers used by the boss attack patterns in `BOSS_PATTERNS`.
10. **Boss logic**: `spawnBoss`, `updateBoss`, `killBoss`.
11. **Rendering**: `draw()` renders each frame, with helpers such as `drawPlane`, `drawBossShip`, and `drawClouds`.
12. **Main loop**: `loop(now)` drives `update` and `draw` through `requestAnimationFrame`. The loop starts at the bottom of the script, and the start button calls `resetGame()`.

Game state lives in module-level `let` variables inside the IIFE. Nothing is exported to `window`.

## Conventions

- Keep the game in a single file unless the user asks for a split. The current design depends on that being simple to open and share.
- Match the existing style: 2-space indentation, single quotes, `const` by default, arrow functions for callbacks, and `// ----------  Section ----------` banner comments to separate sections.
- Add new tuning values to the existing tables instead of hard-coding numbers in logic.
- Respect the reduced-motion setting (`reducedMotion`) for any new screen shake, flash, or pulse effect.
- Keep the game accessible: announce important state changes through `announce()`, which writes to the `srLive` live region.
- Keep the mobile layout working. Test any HUD or control change at phone width as well as desktop width.

## Verifying changes

There is no automated test suite or linter. To verify a change:

1. Check that the script still parses. For example, open the page in a browser and watch the console for errors.
2. Play through at least one stage on each difficulty you touched, including a boss fight if you changed combat, power-ups, or bosses.
3. Check the mute, pause, reduced-motion, and difficulty controls if you changed code near them.
4. For visual or layout changes, take screenshots at desktop and phone widths. A pre-installed Chromium with Playwright is available in the cloud environment.

State clearly in the final message which of these you ran and which you could not run.

## Git workflow

- Develop on the branch the session specifies. The current development branch is `claude/gifted-tesla-blxb7w`.
- Use clear, descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless the user asks for one.
- Do not commit generated or local files. The repo currently has no `.gitignore`, so check `git status` before staging.
