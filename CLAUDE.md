# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas app.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`). ~1,670 lines. |
| `README.md` | Minimal project readme (title only).                           |

There is no build system, no package manager, no test suite, and no dependencies.

## Running the project

- Open `index.html` directly in a browser, or serve the directory with any static server
  (e.g. `python3 -m http.server` and visit `http://localhost:8000`).
- No install or compile step is needed.

## Architecture of `index.html`

- **Single IIFE** inside one `<script>` block, wrapped in `(function () { 'use strict'; ... })();`.
  Keep new code inside this scope; do not add globals.
- **Rendering:** one `<canvas id="gameCanvas">` drawn with the 2D context. A fixed logical
  size (`BASE_W = 480`, `BASE_H = 720`) is scaled to fit the viewport in `resize()`.
- **Game loop:** `requestAnimationFrame(loop)` → `update(dt)` → `draw()`. `dt` is clamped to
  `0.05s`. Gameplay only advances when `running && !paused`.
- **Config tables** near the top of the script drive balance and look:
  - `DIFFICULTIES` (easy/normal/hard): spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - `BOSS_PATTERNS` holds the boss bullet-pattern functions (radial, spread, spiral, cross, homing, etc.).
- **Audio:** WebAudio synthesized sound only (`beep`, `noiseBurst`, the `sfx` object). No audio
  files. Audio context is created lazily via `ensureAudio()` on a user gesture.
- **Persistence:** `localStorage` under keys prefixed `skyfighter.` (`bestScore`, `muted`,
  `difficulty`, `reducedMotion`). Every access should tolerate storage being unavailable.
- **Input:** keyboard (arrows/WASD move, Space fire, X/Shift bomb, P/Esc pause), mouse
  (move to steer, click to fire), and touch (left-side joystick, FIRE and BOMB buttons).
  Space's default is prevented to stop page scrolling.
- **Accessibility:** a `aria-live` region (`srLive`) is updated through `announce()`; a
  reduced-motion toggle (`body.reduced-motion` CSS class) disables flashes, pulses, pop-ups, and entrance animations.
- **Perf:** `trackPerf(dt)` monitors frame time.

## Conventions

- Keep the project as **one self-contained HTML file** unless the user asks for a split.
  Do not introduce a bundler, framework, or npm dependencies without explicit approval.
- Use plain ES2015+ JavaScript (`const`/`let`, arrow functions) matching the existing style:
  2-space indentation, single quotes, semicolons.
- Name new tunables as constants in the existing config tables rather than hard-coding
  magic numbers in logic.
- Any new persisted setting uses the `skyfighter.` key prefix and a `STORAGE_*` constant.
- Any new UI text should be reachable by screen readers; route status messages through `announce()`.
- Mobile must keep working: any new control needs a touch path, not just keyboard/mouse.

## Making changes

- Read the surrounding function before editing; game logic is tightly coupled through shared
  state variables inside the IIFE.
- Verify gameplay changes by actually running the page (browser or Playwright) and checking
  the affected behaviour, since there are no automated tests.
- Balance changes: adjust the `DIFFICULTIES`/boss constants rather than special-casing
  difficulty inside functions.

## Git

- Default development branch for automated work: `claude/gifted-tesla-6g2umo`.
- Commit messages should be short and imperative, with a body for non-trivial changes.
- Do not open pull requests unless explicitly asked.
