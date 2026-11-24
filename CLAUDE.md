# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is one self-contained file, `index.html` (about 1,670 lines: inline `<style>`, HTML markup, and one inline `<script>`). `README.md` is a placeholder with no project documentation.

There is no build step, no package manager, no bundler, and no test suite. Do not add one unless asked.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static file server.
- Controls (also shown in the in-game hint, `#controlsHint`):
  - Keyboard: arrows or WASD to move, Space to fire, X or Shift to drop a bomb, P or Esc to pause.
  - Mouse: move to steer, click to fire.
  - Touch: drag on the left zone to fly, tap FIRE, tap BOMB.

## Code layout (`index.html`)

- **Styles** (`<style>`, roughly lines 10–360): dark color scheme, HUD pills, vignette/flash overlays, and the `heartBeat` animation. Colors are hard-coded hex or `rgba()` values.
- **Markup**: a single `<canvas id="gameCanvas">` plus HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar), the overlay for title/pause/game-over, the touch controls, and an `aria-live` region (`#srLive`) for screen-reader announcements.
- **Script** (`<script>`, starts around line 422): the entire game lives in one IIFE (`(function () { 'use strict'; ... })()`). Nothing is exported and no globals are created. Keep new code inside that IIFE.

Key sections inside the script, in rough order:

| Area | What it holds |
|---|---|
| DOM refs | `const xxxEl = document.getElementById(...)` lookups at the top |
| Config constants | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, and the `EXPLODE_SFX` map. Audio is created lazily on the first user gesture. |
| Adaptive quality | `trackPerf()` moves a `quality` level (0–2) up or down based on a rolling 45-frame average of frame time |
| Game state | `resetGame()`, `update(dt)` (main simulation step), `applyPowerUp()`, `useBomb()`, `fireWeapon()` |
| Spawning and bosses | `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `updateBoss()`, and the `fire*` pattern helpers used by `BOSS_PATTERNS` |
| Rendering | `drawPlane()`, `drawBossShip()`, explosions, screen shake, and the main `loop` driven by `requestAnimationFrame` |
| Input | keydown/keyup on `window`, joystick helpers, touch and mouse handlers for canvas and buttons |

## Persistence

The game uses `localStorage` with these keys (all prefixed `skyfighter.`):

- `skyfighter.bestScore`: high score, number
- `skyfighter.muted`: `'1'` or `'0'`
- `skyfighter.difficulty`: `easy`, `normal`, or `hard`
- `skyfighter.reducedMotion`: `'1'` or `'0'`, falls back to the `prefers-reduced-motion` media query when unset

Reads and writes are done directly with no try/catch. If you add more storage, wrap it so the game still runs when storage is blocked (private windows, disabled site data).

## Conventions

- **Logical resolution**: the canvas uses a fixed logical size, `BASE_W = 480`, `BASE_H = 720`. `resize()` scales it to fit the viewport. Write game logic in these units, not in CSS pixels.
- **Frame-rate independence**: `update(dt)` takes a delta time. Scale movement and timers by `dt`.
- **Difficulty**: per-difficulty tuning lives in `DIFFICULTIES`. Add new tuning values there, not as literals in the logic.
- **Entities**: enemies, bullets, and power-ups are plain objects in arrays. Collision uses `rectsOverlap()`.
- **Accessibility**: respect `reducedMotion` when adding shake, flashes, or other motion effects. Announce important state changes through `announce()`.
- **Styling**: keep HUD colors consistent with the existing palette (gold `#ffd54f` for values, `#7CFC00` for weapon, `#ff8a65` for multiplier).
- **Touch**: keep `touch-action: none` and the existing `preventDefault()` calls on touch controls so the page doesn't scroll or zoom during play.
- **Dependencies**: none. Do not introduce external scripts, fonts, or CDN links without a reason. The page currently works fully offline.

## Making changes

- Make the smallest change that does the job. Because everything is in one file, avoid large reformatting diffs.
- Test changes by opening the game in a browser and playing through the affected path: start, fire, bomb, a boss fight, game over, and the difficulty and mute toggles. Also check the layout at phone width, since touch controls are part of the game.
- Keep the `Sky Fighter` title, meta description, and `theme-color` consistent if you rename or restyle the game.

## Git

- Development branch for this session: `claude/gifted-tesla-53pxh9`. Do not push to other branches without explicit permission.
- Do not create a pull request unless the user asks for one.
