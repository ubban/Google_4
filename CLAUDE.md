# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

The entire game is a single self-contained file, `index.html` (inline `<style>` and one inline `<script>`, drawn on a `<canvas>`). There is no build step, package manager, framework, test suite, or external asset.

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: markup, CSS, and JavaScript. |
| `README.md` | One-line project title. |

To run it, open `index.html` in a browser (or serve the directory with any static server). No install is needed.

## Repository layout of `index.html`

Top to bottom:

1. **`<head>`**: meta tags, theme colour, and an inline SVG favicon. Do not add external stylesheets or scripts without a reason.
2. **`<style>`**: all CSS. Layout is driven by `#container` (fullscreen, centred), `#gameCanvas`, and the HUD (`#hud`, `.stat` pills, `#hearts`, `#bombs`, `#weaponStat`, `#multStat`). Mobile touch controls live in `#touchControls`. Overlays use `#overlay` / `#overlayCard`.
3. **`<body>` markup**: the HUD, boss bar (`#bossBarWrap`), combo/stage banners, top-bar icon buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), a screen-reader live region (`#srLive`), touch controls (`#moveZone`, `#joystickBase`, `#fireBtn`, `#bombBtn`), and the start/pause overlay (`#startBtn`, `.diffBtn`, `#statsRow`).
4. **`<script>`**: one strict-mode IIFE (`(function () { 'use strict'; ... })()`) that holds all game logic. Key sections, in order of appearance:
   - DOM references (`getElementById` lookups at the top).
   - Constants: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
   - Persistence and settings: best score, mute, difficulty, and reduced-motion flags, all in `localStorage`.
   - Audio: a lazily created Web Audio context (`ensureAudio`), `beep`/`noiseBurst` primitives, and the `sfx` table.
   - Rendering helpers: `drawPlane`, `drawBossShip`, cloud and background drawing, and the screen-shake and flash effects.
   - Input: keyboard (`keys[e.code]`), mouse/pointer aim, and a touch joystick with fire and bomb buttons.
   - Game state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score, lives, bombs, stage, chain, and multiplier variables.
   - Game loop: `update(dt)`, `updateBoss(dt)`, `draw()`, and `loop(now)` (driven by `requestAnimationFrame`).

## Gameplay and tuning reference

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), and `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages**: advance after `KILLS_PER_STAGE` (18) kills. Each stage picks a boss from `BOSSES` (`(stage - 1) % BOSSES.length`) and a background from `THEMES`. Boss HP is `70 + stage * 35`, scaled by the difficulty and boss multipliers.
- **Enemies**: `grunt` (stage 1+), `interceptor` (stage 1+, weaving), and `tank` (stage 2+, 4 HP). `ENEMY_TYPES.minStage` gates which types spawn.
- **Weapons**: `vulcan` (default) and `laser`, switched by weapon power-ups. Fire delay is 0.15 s for vulcan and 0.1 s for laser.
- **Scoring**: enemy `score` values, a chain counter with a score multiplier (`chain`, `chainTimer`, `multiplier`), and a persisted best score.
- **Controls**:
  - Move: arrow keys or WASD. Fire: Space (hold) or mouse/touch.
  - Bomb: `X` or Shift. Pause: `P` or Escape.
- **Accessibility**: a reduced-motion toggle disables screen shake, flashes, and pulses. Respect `prefers-reduced-motion` when adding effects. Use `#srLive` for announcements via `announce()`.
- **Performance**: `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low). New effects should degrade under `quality`.

## Conventions

- **Keep it a single file.** Don't split out modules, add a bundler, or add dependencies unless the user asks. Preserve the no-build, open-and-play nature of the project.
- **Vanilla JS and Canvas 2D only.** Match the existing style: `const`/`let`, plain functions, no transpilation.
- **Game constants belong at the top of the script block** (next to `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`). Tune balance there rather than hard-coding numbers inside `update()`/`draw()`.
- **Storage keys are prefixed `skyfighter.`** Add new persisted settings under the same prefix, wrap access in a check so the game still runs when storage is blocked, and register the key as a `STORAGE_*` constant.
- **Keep the layout responsive.** The canvas logic size is fixed at `BASE_W` 480 × `BASE_H` 720; `resize()` only sets its CSS size to fit the viewport (capped at 1.6×). Changes to HUD or overlay CSS must work on phones (`viewport-fit=cover`, safe-area insets) as well as desktop.
- **Keep input symmetric.** Any new control needs keyboard, mouse/pointer, and touch paths where it makes sense, and must not break `touch-action: none` or `user-select: none` on the body.
- **Don't add new UI strings without thinking about screen readers.** Surface important game events through `announce()`.

## Testing and verification

There is no automated test suite or linter in this repo. Verify changes by:

1. Opening `index.html` in a browser and checking the start overlay, all three difficulties, a full stage clear with a boss fight, bomb use, pause/resume, and game over with the best-score update.
2. Checking the mobile layout (narrow viewport or device emulation) for HUD wrapping and touch controls.
3. Toggling mute and reduced motion, and confirming the preference persists across reloads.
4. For JavaScript edits, a quick syntax check is useful, e.g. extract the `<script>` block and run `node --check` on it.

## Git and workflow

- Default branch work for this repo happens on the branch the session specifies (currently `claude/gifted-tesla-i095cy`). Don't push to other branches without explicit permission.
- Write clear, imperative commit messages describing the gameplay or code change (e.g. "Add laser weapon power-up", "Fix boss bar overflow on narrow screens").
- Keep commits focused: one feature or fix per commit where practical.
- Don't open pull requests unless the user explicitly asks.

## Notes for future changes

- Because everything lives in one file, prefer small, targeted edits with the Edit tool over rewriting large sections, and re-read the surrounding code first to keep the game loop's state consistent.
- When changing a stat or spawn rate, check its interaction with difficulty multipliers and the stage progression (`KILLS_PER_STAGE`, boss HP scaling) so the curve stays sane.
