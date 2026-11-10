# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up written as a single self-contained HTML5 canvas page.

| File        | Purpose                                                              |
|-------------|----------------------------------------------------------------------|
| `index.html`| The entire game: inline `<style>`, markup, and one IIFE `<script>`   |
| `README.md` | Placeholder (repo name and a timestamp); no setup instructions       |
| `CLAUDE.md` | This file                                                            |

There is no build system, package manager, bundler, test suite, linter, or CI config in the
repo. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server
  (e.g. `python3 -m http.server`) and visit it.
- There are no automated tests. Verify changes by loading the page and playing: check the
  title overlay, a stage start, shooting, a bomb, a boss fight, pause (`P` / `Esc`), game over,
  and the difficulty selector.
- Check both desktop (keyboard/mouse) and touch layouts when touching the HUD or controls.
  Touch controls are the on-screen joystick, fire button and bomb button.
- Playwright/Chromium is available in the cloud environment if a headless check is needed.

## Architecture of `index.html`

Everything runs inside one `(function () { 'use strict'; ... })()` closure. Keep new code
inside that closure; do not add globals.

Major sections, in file order:

1. **DOM references** - all `getElementById` lookups are at the top of the script.
2. **Persistence keys** - `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`,
   `difficulty`, `reducedMotion`). Any new persisted setting should use the same prefix and
   be read defensively (the existing code assumes storage works; keep reads falsy-safe).
3. **Adaptive quality** - `trackPerf()` samples frame times and steps `quality` 0/1/2 to
   reduce effects when the frame rate drops.
4. **Tuning tables** - `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes),
   `ENEMY_TYPES` (grunt / interceptor / tank), `BOSSES` (six boss designs with HP multipliers).
   Prefer adjusting these tables over hard-coding numbers in logic.
5. **Audio** - Web Audio synthesis only (`ensureAudio`, `beep`, `noiseBurst`); no audio files.
   Audio must be started from a user gesture (`ensureAudio()` is called on input).
6. **Input** - keyboard (`keys[e.code]`; arrows/WASD move, Space fires, `X`/Shift bomb,
   `P`/Esc pause), mouse on canvas, and touch joystick + fire/bomb buttons.
7. **Layout** - `resize()` fits a `BASE_W` x `BASE_H` (480 x 720) logical canvas to the window.
   Game coordinates are in that logical space.
8. **Game state** - `resetGame()` initialises `player`, `bullets`, `enemies`, `enemyBullets`,
   `particles`, `powerUps`, `boss`. `update(dt)` advances the simulation; `draw()` renders;
   `loop(now)` drives both per frame. Keep update and draw separate.
9. **Weapons and power-ups** - `fireWeapon()` (Vulcan / Laser, 3 levels), `applyPowerUp()`
   handling `weapon`, `bomb`, and `life` drops from `maybeSpawnPowerUp()`.
10. **Bosses** - `BOSS_PATTERNS` are bullet-pattern functions (`fireRadial`, `fireAimedSpread`,
    `fireSpiralBurst`, `fireHomingSeeded`, etc.) selected by `updateBoss()`. `killBoss()` advances
    the stage. Boss HP and the boss bar are scaled by difficulty.
11. **Flow** - `loseLife()`, `endGame()`, `togglePause()`, plus overlay/banner helpers
    (`showStageBanner`, `showComboPop`, `flashHit`, `screenShake`).
12. **Rendering** - `drawPlane`, `drawBossShip`, `drawClouds` (parallax), and `draw()`.

### Conventions in this codebase

- Vanilla JS and DOM only. No frameworks, no imports, no external CDN dependencies.
- Keep the whole game in `index.html` unless the user asks for a split. If you split it, keep
  the page loadable by opening the file directly.
- Use `const`/`let`, arrow functions for callbacks, and the existing naming style
  (camelCase functions, UPPER_SNAKE constants).
- Accessibility features exist and should be preserved: the `srLive` aria-live region
  (`announce()`), `prefers-reduced-motion` handling (`reducedMotion`, `.reduced-motion` body
  class), and labelled controls. New UI should have labels and keyboard access.
- Respect `reducedMotion` when adding screen shake, flashes, or other motion effects.
- Keep the mobile layout working: the page must not scroll horizontally, and touch controls
  must stay usable on a phone-sized viewport.

## Making changes

- Make the smallest change that does the job. The file is large (~1,670 lines); prefer
  targeted edits over rewrites and do not reformat unrelated code.
- When balancing gameplay, change the tuning tables first.
- After any change to game logic or rendering, load the page and play a few seconds to check
  for console errors before committing.
- Do not commit generated files, screenshots, or `node_modules`.

## Git

- Develop on the branch the session names (currently `claude/gifted-tesla-p5hf33`). Do not
  push to other branches without explicit permission.
- Commit messages: a short imperative summary line, then a body describing what changed and
  why, in the style of the existing history (e.g. "Add Sky Fighter: ...").
- Do not open a pull request unless the user explicitly asks for one.
