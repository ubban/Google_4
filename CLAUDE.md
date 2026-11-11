# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on a single `<canvas>`.

- `index.html` — the entire game: markup, CSS, and one inline `<script>` (roughly lines 422–1667). There is no build step, no bundler, no package manager, and no test suite.
- `README.md` — a stub title only.

To run it, open `index.html` in a browser (or serve the directory with any static server). Nothing needs to be installed.

## Repository layout and conventions

- Keep the game self-contained in `index.html` unless the user asks for a split. Do not introduce a build system, framework, or npm dependencies without being asked.
- The script is plain JavaScript wrapped in a `(function () { 'use strict'; ... })();` IIFE. Keep the same style (`const`/`let`, arrow functions, no ES modules, no globals).
- Preserve the existing section layout of the script: DOM refs → storage/constants → audio → rendering helpers → input → game state & `update(dt)` → `draw()` → `loop(now)`.

## Key structures inside `index.html`

- **DOM ids**: `gameCanvas`, `hud` (score, best, mult, weapon, hearts, bombs), `bossBarWrap`, `overlay` / `overlayCard` (menus, start, game over), `touchControls` (`moveZone`, `joystickBase`, `fireBtn`, `bombBtn`), `pauseBtn`, `muteBtn`, `motionBtn`, `.diffBtn` buttons.
- **Configuration constants** (tune gameplay here):
  - `DIFFICULTIES` — `easy` / `normal` / `hard`: spawn rates, enemy speed, enemy shoot chance, starting `lives`, starting `bombs`, `bossHpMul`.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — per-stage sky gradients, cloud/sun colors, and names (cycled by stage).
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP); each has size, HP, speed multiplier, score, and `minStage`.
  - `BOSSES` — six bosses, cycled by stage, with colors and `hpMul`.
- **Persistence** (`localStorage`, all wrapped in plain reads/writes):
  - `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
  - Changing these key names breaks existing players' saved data; treat them as a public interface.
- **Performance scaling**: `trackPerf(dt)` samples frame times and sets `quality` (0 full, 1 medium, 2 low). Any new per-frame effect should respect `quality` and `reducedMotion`.
- **Audio**: a lazily created `AudioContext` (`ensureAudio()`), with `beep`, `noiseBurst`, and the `sfx` table. Audio must start from a user gesture; keep calls going through `ensureAudio()`.
- **Game loop**: `update(dt)` advances state, `updateBoss(dt)` handles bosses, `draw()` renders, `loop(now)` drives both via `requestAnimationFrame`. `togglePause()` gates the loop.
- **Input**:
  - Keyboard: arrows/WASD move, Space fires, X or Shift bombs (`useBomb()`), P or Esc pauses.
  - Mouse: move to steer, hold/click to fire on the canvas.
  - Touch: left-side joystick to fly, FIRE and BOMB buttons.
- **Accessibility**: `announce()` writes to the `srLive` live region; keep screen-reader messages in sync with important game events (stage change, bomb use, game over). Respect `prefers-reduced-motion` via `reducedMotion`.
- **Layout**: the canvas is a fixed logical size (`BASE_W = 480`, `BASE_H = 720`) scaled by `resize()` up to 1.6×. Keep HUD and overlays working on phone-sized viewports (`viewport-fit=cover`, safe-area insets).

## Working guidelines

- **Verify changes in a browser.** There are no automated tests or linters. For gameplay or layout changes, load `index.html` in a real browser (desktop and a narrow/touch viewport if layout changed) and play through the affected path. If you cannot run a browser, say so explicitly rather than claiming it works.
- **Check the JavaScript parses** after edits, e.g. extract the inline script and run `node --check` on it, since a single syntax error blanks the whole game.
- **Balance changes**: prefer editing the constants above over changing logic. Note in your summary which difficulty values you changed.
- **Don't break saved state**: keep `localStorage` keys and value formats stable unless a migration is added.
- **Keep the controls hint in sync**: the `#controlsHint` paragraph describes the controls; update it if you add or rebind inputs.
- Stay within the request's scope; avoid restyling or refactoring unrelated parts of `index.html`.

## Git

- Development branch for this task: `claude/gifted-tesla-igbm8p`. Commit and push there with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
