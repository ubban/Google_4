# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single-file HTML5 canvas game.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`). ~1,670 lines. |
| `README.md` | Placeholder (title only).                                      |

There is no build system, package manager, test suite, or linter configured.

## Running the game

No build step. Open `index.html` directly in a browser, or serve the directory with any
static server (for example `python3 -m http.server` from the repo root) and visit
`http://localhost:8000/`.

## Architecture of `index.html`

Everything lives in one IIFE (`(function () { 'use strict'; ... })();`) inside the
final `<script>` tag. Keep new code inside that scope rather than adding globals.

Top-to-bottom layout of the file:

1. **`<head>`** – meta tags (viewport locked, `theme-color` `#0b1d3a`) and `<style>`
   (HUD, overlay card, touch controls, animations).
2. **`<body>`** – HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar),
   overlay/menu, touch controls, and a single `<canvas id="gameCanvas">`.
3. **Constants** – `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE`,
   `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, and `BASE_W`/`BASE_H` (480×720 logical
   canvas size).
4. **Audio** – `ensureAudio()` lazily creates the `AudioContext`; `beep()` and
   `noiseBurst()` synthesize all sound effects (no audio files).
5. **Game state & systems** – `resetGame()`, `update(dt)` (per-frame simulation),
   spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`,
   `applyPowerUp`), bombs (`useBomb`), scoring/kill chain (`addScore`, `breakChain`),
   and life/game-over flow (`loseLife`, `killBoss`, `endGame`, `togglePause`).
6. **Boss patterns** – `BOSS_PATTERNS` is an array of bullet-pattern functions
   (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`,
   `fireHomingSeeded`, `fireDoubleRing`); `updateBoss(dt)` drives them.
7. **Rendering** – `draw()` renders the frame; helpers include `drawPlane`,
   `drawBossShip`, `drawClouds` (parallax), explosions, and screen shake.
   `drawPlane` in particular is tuned for the look of the player and enemy ships.
8. **Input** – keyboard (`keys[e.code]`), mouse/pointer via `canvasPointFromEvent`, and a
   touch joystick plus fire/bomb buttons (`joystickStart/Move/End`).
9. **Main loop** – `loop(now)` caps `dt` at 0.05 s, calls `update` and `draw` while
   running and not paused, then re-queues itself with `requestAnimationFrame`.

### Controls (for reference when changing input code)

- Move: arrow keys or WASD (desktop); mouse position over the canvas (`mouseTarget`, `mouseActive`); joystick on touch devices.
- Fire: `Space` (held) or the fire button.
- Bomb: `X`, `Shift`, or the bomb button.
- Pause: `P` or `Escape`.
- Mute, motion toggle, and difficulty are controlled from the HUD/menu.

### Persistence

Settings and scores use `localStorage` under these keys (keep the `skyfighter.` prefix):

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads are not wrapped in try/catch today; if you touch that code, consider guarding it
because storage can be unavailable in private browsing or when site data is blocked.

## Conventions

- **Single file.** Keep the game self-contained in `index.html`. Do not introduce
  external libraries, bundlers, or asset files unless the user asks for that change.
- **No external assets.** Graphics are drawn with canvas primitives and sound is
  synthesized with the Web Audio API. Preserve that approach.
- **Fixed logical resolution.** Game coordinates use the 480×720 `BASE_W`/`BASE_H`
  space; `resize()` scales the canvas for display. Position and size new entities in
  those units.
- **Time-based updates.** Movement and timers multiply by `dt` (seconds). Do not
  hard-code per-frame increments.
- **Difficulty is data-driven.** Tune values in `DIFFICULTIES`, `ENEMY_TYPES`, and
  `BOSSES` rather than adding branches in the update logic.
- **Accessibility hooks exist.** Respect `reducedMotion` for shake/flash effects, and use
  the `srLive` live region (`announce()`) for screen-reader announcements.
- **Style.** Match the existing 2-space indentation, single quotes in JS, `const`/`let`,
  and arrow functions for callbacks.

## Verifying changes

There are no automated tests. After changing game code:

1. Open `index.html` in a browser (or serve it locally) and check the browser console
   for errors.
2. Play through: start, move, fire, use a bomb, pause/resume, lose a life, reach game
   over, and confirm the best score persists after reload.
3. For layout or HUD changes, check both a desktop viewport and a phone-sized viewport
   (touch controls appear there).

## Git

- Work on the feature branch named in the session instructions; do not push to other
  branches without permission.
- Commit messages in this repo use a short imperative summary line followed by a body
  that explains the change in prose or bullets.
