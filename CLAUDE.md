# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser. It is a single self-contained static page:

- `index.html` (~1,670 lines): all HTML, CSS (inline `<style>`), and JavaScript (inline `<script>`, wrapped in one IIFE). No build step, no bundler, no dependencies, no external assets. Audio is synthesized with the Web Audio API.
- `README.md`: a placeholder title only.

There is no package.json, test suite, linter, or CI configuration in the repo.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a run, check movement, firing, bombs, pause, stage transitions, boss fights, game over, and best-score persistence. Test at both desktop and narrow/mobile viewport sizes if you touched layout.

## Code layout inside `index.html`

- **CSS** (~lines 10–358): theme colors (dark `color-scheme`), HUD, stage banner, overlay/menus, touch controls, and a `body.reduced-motion` class that disables animations.
- **Markup** (~lines 359–421): canvas `#gameCanvas`, HUD (`#hud`, score/best/mult/weapon/hearts/bombs), overlay with start/difficulty controls, touch controls (`#touchControls`, joystick, FIRE and BOMB buttons), and an `aria-live` region `#srLive` for screen readers.
- **Script** (~lines 422–1667), grouped roughly as:
  - DOM refs and `STORAGE_*` keys (`localStorage` keys are prefixed `skyfighter.`).
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object.
  - HUD helpers: `renderHearts`, `renderBombs`, `flashHit`, `showStageBanner`, `screenShake`, etc.
  - Config tables: `DIFFICULTIES` (~485), `ENEMY_TYPES` (~502), `BOSSES` (~508), `THEMES` (~493), `KILLS_PER_STAGE = 18`, `BASE_W × BASE_H = 480 × 720`.
  - Input: keyboard listeners (~769), mouse/pointer, and virtual joystick for touch.
  - Game state and lifecycle: `resetGame`, `update(dt)`, `draw()`, `loseLife`, `killBoss`, `endGame`, `togglePause`.
  - Entities and weapons: `spawnEnemy`, `spawnBoss`, `fireWeapon`, `applyPowerUp`, `useBomb`, and the boss attack patterns (`BOSS_PATTERNS`, ~1372).
  - Main loop: `loop(now)` drives `update` and `draw` via `requestAnimationFrame`.

## Game rules worth knowing

- Controls: mouse steers and clicks to fire; Arrows/WASD move; Space fires; X or Shift drops a bomb; P or Esc pauses. Touch devices get a left-side drag joystick plus FIRE and BOMB buttons.
- Difficulty (`easy` / `normal` / `hard`) sets spawn rates, enemy speed and fire chance, starting lives, starting bombs, and boss HP multiplier. The selection persists in `localStorage`.
- Each stage ends after `KILLS_PER_STAGE` kills, then a boss fight begins. Boss HP scales with stage number and the boss design's `hpMul`.
- Stages cycle through six `THEMES` and six boss designs by index (`(stage - 1) % length`).
- Weapons toggle between `vulcan` and `laser`. Power-ups can add lives (capped at `maxLives`).
- Best score is stored under `skyfighter.bestScore`.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Don't add a build tool, framework, or external CDN dependency without asking.
- Preserve the `'use strict'` IIFE structure and the existing naming style (camelCase functions, UPPER_CASE config constants).
- Keep the HUD and overlay accessible: update `#srLive` through `announce()` for significant state changes, and honor the reduced-motion toggle when adding animations.
- Keep touch support working. Any new control needs a touch path as well as keyboard and mouse.
- Keep layout responsive: the canvas scales to the viewport with a 480×720 logical resolution, and `touch-action: none` is set on the page.
- Don't change `localStorage` key names without a migration plan, since they hold players' saved best score and settings.

## Git

- Development branch for this work is `claude/gifted-tesla-6x1gu6`. Commit and push there; don't push to other branches without permission.
- Use descriptive commit messages in the imperative mood, matching the existing history style.
