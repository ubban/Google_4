# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines): inline `<style>`, the markup, and one inline `<script>` IIFE. There is no build step, no bundler, no package manager, and no external dependencies or CDN scripts.

Files:

- `index.html` — the whole game (markup, CSS, JS).
- `README.md` — placeholder title only.

## Running and testing

- Run: open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There is no test suite, linter, or build. Verify changes by loading the page and playing: check desktop controls, touch controls (or device emulation), pause, bombs, stage transitions, boss fights, and game over.
- When changing gameplay numbers, confirm the HUD and overlay text still match (see "Gotchas").

## Code layout inside `index.html`

- **Markup** (~lines 362–420): `#container` holds `#gameCanvas`, the HUD (`#hud`, `#topBar`), boss bar (`#bossBarWrap`), combo/stage popups, touch controls (`#touchControls`), and `#overlay` (start/pause/game-over card with `#statsRow` and `#diffRow`).
- **CSS** (~lines 10–360): design tokens are not factored out; colors are literal values. Mobile layout and reduced-motion rules (`body.reduced-motion`) are here.
- **Script** (~lines 422–1668): one IIFE with `'use strict'`. Sections, in order:
  - DOM references and `STORAGE_*` keys.
  - Adaptive quality (`trackPerf`, `quality` 0–2) that adjusts rendering cost based on frame times.
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` map. Audio is lazily created on the first user gesture.
  - UI helpers: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
  - Input: keyboard, mouse/pointer, and virtual joystick and buttons for touch.
  - Game state and loop: `resetGame`, `update(dt)`, `draw()`, `loop(now)`. `dt` is clamped to 50 ms.
  - Gameplay: spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`), bombs (`useBomb`), power-ups (`applyPowerUp`), scoring and kill chain (`addScore`, `breakChain`), boss attack patterns, `killBoss`, `loseLife`, `endGame`, `togglePause`.
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`.

## Game rules (current behavior)

- Controls: mouse moves and clicks to fire; Arrows/WASD move; Space fires; X or Shift bombs; P or Esc pauses. Touch: left-side drag flies, FIRE and BOMB buttons.
- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy shot chance, starting lives, starting bombs, and boss HP multiplier.
- Enemies: `grunt` (stage 1+), `interceptor` (stage 1+, weaves), `tank` (stage 2+, 4 HP).
- Each stage needs `KILLS_PER_STAGE` (18) kills, then a boss. Boss HP is `(70 + stage * 35)` scaled by the boss's `hpMul` and the difficulty multiplier. Bosses cycle through `BOSSES` by `(stage - 1) % BOSSES.length`.
- Power-ups: `life`, `bomb`, `weapon` (Vulcan/Laser, 3 levels).
- Persistence: `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. All accesses are unguarded; if you add more, consider wrapping them in try/catch since storage can be blocked.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not add a build step or framework without asking.
- Match the existing style: 2-space indent, single quotes, `const`/`let`, arrow functions for callbacks, section headers written as `// ---------- Name ----------`.
- Tuning numbers live in the constant tables near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer editing those over hard-coding values in logic.
- Respect `reducedMotion` for any new screen shake, flash, or motion effect, and keep the screen-reader announcements (`announce`) for important state changes such as life loss and stage change.
- Support both keyboard/mouse and touch when adding a new input action.

## Gotchas

- The HUD, overlay text, and `controlsHint` (in the markup) describe the controls and rules by hand. Update them when controls or rules change.
- `dt` is capped at 50 ms, so long frames (tab switches) do not teleport objects. Prefer timers in `dt` units over frame counts.
- Spacebar scrolling is suppressed by `preventDefault` in the keydown handler; keep that when touching input code.
- The best score is written to `localStorage` only at game over (`endGame`), and only when the score beats the stored best. Quitting mid-run does not save it.

## Git and workflow

- Default working branch for automated work: `claude/gifted-tesla-hc7h46`. Commit with clear messages and push with `git push -u origin <branch>`.
- Do not open pull requests unless asked.
