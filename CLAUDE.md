# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static repository. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-file HTML5 canvas game.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`). About 1,670 lines. |
| `README.md` | Minimal project title. |

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Architecture of `index.html`

The game lives in one IIFE (`(function () { 'use strict'; ... })();`) inside the inline script. Keep new code inside that closure rather than adding globals.

Major sections, roughly in file order:

- **DOM references**: `const xxxEl = document.getElementById(...)` block at the top of the script. Add new HUD or overlay elements here.
- **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`. Best score, mute, difficulty, and reduced-motion settings persist via `localStorage`.
- **Tuning tables**: `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`. Balance changes usually belong here.
- **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, `panFor()`. Audio is Web Audio API based and created lazily on first user gesture.
- **Input**: keyboard handlers (`keydown`/`keyup` on `window`), pointer and touch handling for the joystick and fire/bomb buttons, and `canvasPointFromEvent()` for coordinate mapping.
- **Responsive scaling**: `resize()` fits a logical `BASE_W` x `BASE_H` (480 x 720) play field to the viewport.
- **Game state and simulation**: `resetGame()`, `update(dt)`, `spawnEnemy()`, `spawnBoss()`, `fireWeapon()`, `useBomb()`, `applyPowerUp()`, `updateBoss()`, `killBoss()`, `loseLife()`, `endGame()`.
- **Rendering**: `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`. Rendering is canvas-only; the HUD (hearts, bombs, score, boss bar) is DOM.
- **Main loop**: `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05 s, and only calls `update` and `draw` while the game is running and not paused.

Controls, for reference:
- Keyboard: arrow keys or WASD to move, Space to fire, X or Shift to use a bomb, P or Escape to pause.
- Touch: on-screen joystick, fire button, and bomb button.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not introduce a build system or dependencies without asking.
- **No external dependencies.** The game uses only browser APIs (Canvas 2D, Web Audio, `localStorage`). Do not add CDN scripts or libraries unless explicitly requested.
- **Keep the IIFE and `'use strict'`.** Do not leak variables to `window`.
- **Put tunable numbers in the tuning tables** rather than hard-coding them in update or draw logic.
- **Respect the reduced-motion and mute settings.** Screen shake, flashes, and audio should consult the existing `reducedMotion` and `muted` state.
- **Preserve accessibility hooks.** The `srLive` live region is used for announcements via `announce()`; use it for new important status messages.
- **Mobile and desktop both matter.** Any new control must work with both keyboard and touch input.

## Verifying changes

No automated tests exist. To verify a change:

1. Open `index.html` in a browser (or serve it locally as above).
2. Play through at least one stage and one boss fight on each difficulty you touched.
3. Check the browser console for errors.
4. For UI or layout changes, check both a desktop viewport and a phone-sized viewport.

If you add tests or tooling, document the commands here.

## Git workflow

- Branches: `main` holds the baseline. Session work is developed on `claude/gifted-tesla-09avbe`.
- Commit messages: a short imperative summary line, then a bulleted body describing the change, matching the style of the existing `Add Sky Fighter...` commit.
- Do not force-push, and do not open pull requests unless the user asks.

## Notes for future edits

- The game loop, input, audio, and rendering are all in one file, so search by function name (for example `grep -n "function drawBossShip" index.html`) instead of scrolling.
- Changes to persistence keys break saved player data. Avoid renaming `STORAGE_*` values unless a migration is intended.
