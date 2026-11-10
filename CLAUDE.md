# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only app is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in the browser as a single
self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript in one file (~1,670 lines). |
| `README.md` | Placeholder (title only). |

There is no build system, package manager, bundler, test suite, or linter.
Nothing needs to be installed. To run the game, open `index.html` in a browser
or serve the directory with any static file server.

## Repository and branches

- Default work branch for this task series: `claude/gifted-tesla-258hj4`.
- Commit with clear, descriptive messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.

## Architecture of `index.html`

The file is organized into three sections.

1. **`<style>`** (around lines 10–362): CSS for the full-viewport
   container, the canvas, the pill-style HUD, glassmorphic overlay card,
   on-screen touch controls, and the `.reduced-motion` class. The page is locked
   to the viewport (`overflow: hidden`, `touch-action: none`).
2. **`<body>` markup** (around lines 363–421): `#gameCanvas`, the HUD
   (`#hud`, mute / reduced-motion / pause icon buttons), touch controls
   (`#moveZone` joystick, `#fireBtn`, `#bombBtn`), and `#overlay` / `#overlayCard`
   holding the title, difficulty buttons (`.diffBtn`, `data-diff`), `#startBtn`,
   and `#controlsHint`.
3. **`<script>`** (starts around line 422): one `(function () { 'use strict'; ... })();`
   wrapper with no modules. Key sections, in source order:
   - Constants: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`
     (six stage palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES`
     (six bosses).
   - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is
     synthesized with the Web Audio API; there are no audio files.
   - HUD and feedback helpers: `renderHearts`, `renderBombs`, `flashHit`,
     `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
   - Input: keyboard (`keys` map), touch joystick, fire/bomb buttons, and mouse
     steering on the canvas.
   - Game state and loop: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`,
     `maybeSpawnPowerUp`, `spawnExplosion`, `addScore` / `breakChain` (kill-chain
     multiplier), `useBomb`, `fireWeapon`, `applyPowerUp`.
   - Boss logic: `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross,
     homing, double ring) driven by `updateBoss`, and `killBoss`.
   - Rendering: `drawPlane`, `drawBossShip`, and the background / parallax
     layers, drawn to the canvas each frame.
   - Persistence: best score, mute, difficulty, and reduced-motion preference
     are stored in `localStorage` under `skyfighter.*` keys.

### Gameplay rules to keep in mind

- **Difficulty** (`DIFFICULTIES`): easy, normal (default), hard. Each sets spawn
  rates, enemy speed, enemy shoot chance, starting lives, starting bombs, and a
  boss HP multiplier.
- **Stages**: each stage lasts `KILLS_PER_STAGE` (18) kills, then a boss fights.
  Stage number selects a `THEMES` entry and a `BOSSES` entry, both cycling
  with `(stage - 1) % length`.
- **Adaptive quality**: a frame-time tracker (`trackPerf`) steps rendering
  quality down (full → medium → low) when the frame rate drops.
- **Weapons**: `vulcan` and `laser`, each with three levels. Picking up a weapon
  power-up raises the level; past level 3 it toggles to the other weapon and
  resets to level 1.
- **Power-ups**: `life`, `bomb`, `weapon`. Drops come from `maybeSpawnPowerUp`.
- **Bombs** are limited (`bombCount` / `maxBombs`) and are triggered by X,
  Shift, or the BOMB button. They clear the screen and damage bosses.
- **Pause** is P or Esc, or the pause button.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks
  for a restructure. Do not add a build step, framework, or external
  dependencies without an explicit request.
- Match the existing style: 2-space indentation, `const` / `let`, arrow
  functions for callbacks, and `UPPER_SNAKE_CASE` for constant tables.
- Game tuning values live in the constant tables near the top of the script
  (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer changing those
  over hard-coding numbers in logic.
- Keep the page mobile-friendly: preserve the touch controls, the viewport
  lock, and the `safe-area-inset` padding in the HUD.
- Preserve accessibility hooks: the reduced-motion toggle and the `announce()`
  live region (`#srLive`, `role="status"`).
- Persisted keys use the `skyfighter.` prefix. Do not rename them without a
  migration, because that would reset players' saved best score and settings.

## Verification

There is no automated test suite. To check a change:

1. Open `index.html` in a browser (or serve the folder statically).
2. Start a game on each difficulty and confirm the HUD, weapons, bombs, and
   pause behave as described above.
3. Reach the first boss (18 kills) and confirm the boss fight and stage banner.
4. Check the layout at phone width with touch controls, and at desktop width
   with the mouse and keyboard.
5. Run a JavaScript syntax check on the inline script if you have Node.js:
   extract the contents of the `<script>` block into a temp file and run
   `node --check` on it.

## Notes for future changes

- `README.md` is a stub. If you document the project further, update it there
  and keep this file focused on guidance for working in the code.
- Nothing in the repo is generated, so edits go directly to `index.html`.
