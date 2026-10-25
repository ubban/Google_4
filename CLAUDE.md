# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** is a single-page, Raiden-style vertical shoot-em-up built on an HTML5 `<canvas>`. The entire game is one file, `index.html`, with inline CSS and one inline `<script>` block wrapped in an IIFE (`'use strict'`).

- No build step, bundler, package manager, or test suite. There is no `package.json`.
- No external scripts, fonts, or network requests. Keep it that way unless the user asks otherwise.
- `README.md` is a placeholder (one line). Don't treat it as the source of truth for behavior.

To run it, open `index.html` in a browser. There is nothing to install or compile.

## Layout of `index.html`

| Section | Approx. lines | Contents |
| --- | --- | --- |
| `<style>` | 10–360 | All CSS: HUD pills, overlay card, boss bar, touch controls, reduced-motion styles |
| Markup | 361–421 | `#container`, `#gameCanvas`, HUD (`#hud`, `#topBar`), boss bar, touch controls, `#overlay` start/pause/game-over card |
| Script | 422–1665 | Game logic, grouped roughly as below |

Within the script, the major groups in order are: DOM refs and constants → persistence and settings → audio (WebAudio, `beep` / `noiseBurst`) → HUD rendering and effects → resize and input (keyboard, joystick, mouse) → game state and `resetGame` → drawing (`drawPlane`, `drawBossShip`) → spawning (enemies, bosses, power-ups) → `update(dt)` → weapon patterns (`fire*` helpers) → `loop(now)` at the bottom, driven by `requestAnimationFrame`.

Search for these names to navigate: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`, `resetGame`, `update`, `applyPowerUp`, `spawnBoss`, `loop`.

## Game design constants

Balance numbers live in the constant tables near the top of the script. Change them there, not inline.

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy shot chance, starting lives and bombs, and a boss HP multiplier.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaves), `tank` (4 HP, appears from stage 2).
- **Bosses** (`BOSSES`): the boss for each stage is `BOSSES[(stage - 1) % 6]`, so the six bosses cycle. Each has its own palette and HP multiplier.
- **Stages**: `KILLS_PER_STAGE` kills advance the stage. The background theme is `THEMES[(stage - 1) % 6]`.
- **Power-ups**: `life`, `bomb`, and `weapon` (raises weapon level).
- **Scoring**: kill-chain multiplier (`chain`, `chainTimer`, `multiplier`). Breaking the chain resets it.

## Input

- **Keyboard**: arrow keys or WASD to move. Space fires (hold). `X`, `Shift` drop a bomb. `P` or `Escape` toggles pause. Space's default is prevented so the page doesn't scroll.
- **Touch**: left-side joystick (`#moveZone`), right-side fire and bomb buttons (`#fireBtn`, `#bombBtn`).
- **Mouse**: canvas mouse movement steers the ship, and mousedown fires.

## Persistence

Settings and scores are stored in `localStorage` under these keys. Keep the `skyfighter.` prefix and the `try/catch` style if you add more.

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reading from storage is currently unguarded. If you touch that code, consider wrapping it, since storage access can throw in private windows or when site data is blocked.

## Performance and accessibility

- `trackPerf(dt)` samples the last 45 frames and steps `quality` (0 full, 1 medium, 2 low) down when the average is slow (under ~40 fps) and back up when it's fast (over ~56 fps). Gate expensive visuals on `quality` when adding effects.
- Reduced motion (`reducedMotion`) disables screen shake and similar effects. Respect it for any new motion.
- Screen-reader announcements go through `announce()`, which writes to the `#srLive` live region. Use it for important state changes.
- Layout must work on small screens. The canvas scales via `resize()` against `BASE_W` × `BASE_H` (480 × 720).

## Working conventions

- **Edit in place.** Keep changes inside the existing IIFE and match the file's style: 2-space indent, single quotes in JS, `const`/`let`, short helper functions, and brief comments only where the logic isn't obvious.
- **Verify in a browser.** There are no automated tests. For gameplay or UI changes, open `index.html` in a browser (or use a headless Chromium with Playwright) and check the start screen, a full run through a stage transition, pause/resume, game over, and mobile layout (touch emulation).
- **Watch the loop.** `update(dt)` is frame-rate sensitive. Use `dt` for movement and timers, never a fixed per-frame step, and keep game logic out of the draw functions.
- **Don't introduce a build system or dependencies** without asking. A single self-contained file is a deliberate choice here.
- **Commits.** Use short imperative subjects that describe the gameplay or UI change (the existing history uses this style, e.g. "Add Sky Fighter: …").
- Don't create pull requests unless the user asks.
