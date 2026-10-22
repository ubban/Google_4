# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (inline `<style>` and `<script>`, no build step, no dependencies). `README.md` is a placeholder.

There is no package manager, test suite, linter, or build. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

- `index.html` — the whole game: markup for the HUD and overlays, CSS, and the game engine in one IIFE.
- `README.md` — minimal project readme.
- `CLAUDE.md` — this file.

## Code structure of `index.html`

The script is wrapped in a single `(function () { 'use strict'; ... })();` and organized roughly as:

1. **DOM references and storage keys.** Elements are looked up once at the top. Persisted settings use `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every storage read/write should tolerate failure the same way the existing code does.
2. **Tuning tables** (plain constants; edit these to rebalance rather than hard-coding numbers elsewhere):
   - `DIFFICULTIES` — `easy` / `normal` / `hard`, controlling spawn timing, enemy speed, enemy fire chance, starting `lives`, starting `bombs`, and `bossHpMul`.
   - `KILLS_PER_STAGE` — kills needed before a stage boss is triggered.
   - `THEMES` — sky gradient, cloud/sun colors, and name per stage; indexed by `(stage - 1) % THEMES.length`.
   - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank`, with size, HP, speed multiplier, score, and `minStage` gating.
   - `BOSSES` — boss designs cycled by `(stage - 1) % BOSSES.length`; boss HP scales as `70 + stage * 35` times the design and difficulty multipliers.
3. **Adaptive quality.** `trackPerf` averages the last 45 frame times and steps a `quality` level down when the frame rate drops below ~40 fps and back up above ~56 fps. Visual effects should respect this level, not bypass it.
4. **Input.** Keyboard (arrows/WASD to move, Space to fire, X or Shift for bomb, P/Esc to pause), mouse (move to steer, click to fire), and touch (left-side joystick, FIRE and BOMB buttons). New controls should be wired into all three paths where they make sense.
5. **Audio.** An `AudioContext` is created lazily by `ensureAudio()` on the first user gesture; sounds are suppressed when `muted` is set.
6. **Game state and loop.** `update(dt)` advances the simulation (spawning, movement, collisions, weapons, boss patterns); `draw()` renders. `loop(now)` drives both via `requestAnimationFrame`, clamps `dt` to 50 ms, and skips update/draw while paused or not running.
7. **Weapons, bombs, and bosses.** `fireWeapon()` handles the Vulcan/Laser system (3 power levels). `useBomb()` clears the screen and damages bosses. Boss attacks are separate `fire*` pattern helpers (radial, spread, columns, spiral, cross, homing, double ring) selected by `updateBoss(dt)`.
8. **Progression and game over.** Stages advance after a boss is defeated (`stage++`), with a stage banner and a screen-reader announcement. Game over updates the persisted best score and shows the overlay stats.

Game state is kept in top-level `let` variables inside the IIFE (e.g. `score`, `lives`, `stage`, `running`, `paused`, `gameOver`). There are no modules, classes, or globals to worry about.

## Conventions

- **Keep it one file.** Do not split `index.html` into separate JS/CSS files or add a build step unless the user asks. New code goes in the existing `<script>` block, near related code.
- **Match the existing style.** 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short helper functions named with verbs (`spawnEnemy`, `fireWeapon`, `updateBoss`).
- **Tune via the tables.** Prefer adding or changing entries in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` over introducing new magic numbers.
- **Accessibility is part of the feature.** The game has a reduced-motion toggle (screen shake, flash, and pulse), a `#srLive` live region for announcements (use `announce()`), and labelled buttons. Keep new effects behind the reduced-motion setting and announce significant state changes.
- **Mobile matters.** The layout uses `touch-action: none`, safe-area insets, and responsive `clamp()` sizes. Check new UI at phone width and with touch input.
- **Don't add dependencies.** No external scripts or fonts; the page should keep working offline from a single file.

## Verifying changes

There is no automated test setup. To verify a change:

1. Open `index.html` in a browser (desktop and, if the change touches input or layout, a mobile-sized viewport).
2. Start a game on each difficulty you affected, and exercise the feature directly: movement, firing, bombs, pause/resume, mute, reduced-motion toggle, and game over.
3. Check the browser console for errors. Note that `'use strict'` is in effect, so typos in variable names fail loudly.
4. Confirm that best score, mute, difficulty, and reduced-motion settings persist across a reload.

If you cannot run a browser, say so explicitly and verify by reading the code paths you changed.

## Git

- Development happens on feature branches named `claude/...`; do not push to other branches without permission.
- Write commit messages in the imperative mood with a short summary line, and describe *why* in the body when it isn't obvious (the existing history follows this pattern).
