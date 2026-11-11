# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based, Raiden-style vertical shoot-'em-up rendered on a `<canvas>`.

- `index.html` — the entire game: inline CSS, markup, and one inline `<script>` block (~1,670 lines). No external assets, libraries, or network calls.
- `README.md` — minimal placeholder (title only).

There is no build step, package manager, bundler, test suite, or linter. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Working conventions

- Keep the game in `index.html` unless the user asks for a restructure. Do not introduce a build system, framework, or external CDN dependencies without an explicit request.
- Preserve the zero-dependency, no-network nature of the page.
- Make small, targeted edits. Do not reformat or reorganize unrelated code.
- Verify changes by loading the page in a browser (desktop and a narrow/mobile viewport) and exercising the affected behavior. There are no automated tests to run.
- Check that the JavaScript parses before committing, e.g. by extracting the `<script>` block and running `node --check` on it.

## Code layout (inside `index.html`)

All game code lives in one `(function () { 'use strict'; ... })();` wrapper inside the page's `<script>` tag (starts ~line 423, ends ~line 1666). Code is organized by section:

- **DOM refs** (~lines 426–461): cached element lookups by id. Keep these in sync when changing markup.
- **Persistence** (~lines 463–466): `localStorage` keys are prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes should stay defensive (values are strings; `Number(...) || 0` pattern for best score).
- **Config tables** (~lines 485–515):
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18) controls stage progression.
  - `THEMES` (6 sky palettes, cycled by stage).
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2). Each has `minStage`.
  - `BOSSES` (6 bosses, indexed by `(stage - 1) % BOSSES.length`).
- **Audio** (~lines 543–640): lazily created `AudioContext` via `ensureAudio()`; `beep`, `noiseBurst`, and a `sfx` map. Audio must only start after a user gesture.
- **HUD** (~lines 680–740): `renderHearts`, `renderBombs`, combo pops, stage banners, screen shake, vignette/flash, mute and difficulty controls.
- **Scaling** (~lines 753–762): fixed logical canvas of `BASE_W × BASE_H` = 480 × 720, scaled to fit the window (capped at 1.6×).
- **Input** (~lines 768–862):
  - Keyboard: arrows/WASD move, `Space` fires, `X` or `Shift` bombs, `P`/`Esc` pauses.
  - Mouse: move to steer, click to fire on the canvas.
  - Touch: left-side joystick (`moveZone`), `fireBtn`, `bombBtn`.
- **Game state** (~lines 864–911): module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`/`multiplier`, etc.). `resetGame()` reinitializes them.
- **Rendering** (~lines 913–1147 and `draw()` ~line 1520): `drawPlane`, `drawBossShip`, cloud and background drawing. Rendering is separate from update logic.
- **Simulation** (~lines 1103–1410): `useBomb`, `update(dt)`, `updateBoss(dt)`, enemy/bullet/power-up spawning and collision.
- **Loop** (~lines 1491–1669): `togglePause`, `draw`, `loop(now)` driven by `requestAnimationFrame`, and the start button handler.

Quality scaling: `trackPerf(dt)` tracks frame times and sets a `quality` level (0 full, 1 medium, 2 low). Keep new visual effects compatible with this — gate expensive effects on `quality` where practical.

Reduced motion: the `reducedMotion` flag (user toggle, defaulting to `prefers-reduced-motion`) should disable or soften screen shake, flashes, and similar effects. New motion-heavy effects must respect it.

## Gameplay invariants to keep in mind

- Logical coordinates are always 480 × 720; rendering is scaled, so do not hard-code physical pixel sizes.
- Difficulty can only be changed between runs (the buttons are ignored while a game is running and not paused); `setDifficulty` persists the choice to `localStorage`.
- The best score is written to `localStorage` only at game over, and only when the final score exceeds it.
- Bomb use is gated by `bombCount`; the HUD must reflect `maxBombs`/`bombCount`.
- Weapon state lives on `player.weapon` (`'vulcan'` or `'laser'`) with `player.level`.

## Accessibility and UX

- The page uses `aria-live` (`srLive`) for announcements via `announce()`. Use it for significant state changes (pause, game over, stage change, new best).
- Touch controls must remain usable on phones: the viewport is locked (`user-scalable=no`), `touch-action: none` is set, and the `touchControls` layer is enabled (`.enabled` class) on touch devices.
- Keep the overlay/menus and HUD usable at phone width without horizontal page scroll (the body is `overflow: hidden`, so layout overflow is easy to miss).

## Git and branches

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-84dpgy`). Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages in the imperative mood, e.g. `Add boss telegraph for Titan`.
- Do not create pull requests unless explicitly asked.
