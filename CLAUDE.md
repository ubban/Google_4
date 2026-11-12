# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game, delivered as a single self-contained HTML5 canvas page. The repo is small: there is no build step, no bundler, no package manifest, and no test suite.

Files:

- `index.html` — the entire game: markup (~lines 1–360 CSS in `<style>`, ~361–421 body/HUD/overlay markup), and one IIFE `<script>` (~lines 422–1667) containing all game logic, rendering, audio, and input.
- `README.md` — minimal project readme.

## Running it

No build or install is needed. Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and browse to it.

There are currently no automated tests or linters configured. Verify changes by playing the game in a browser, checking the console for errors, and exercising both desktop (keyboard/mouse) and touch controls.

## Architecture (inside the game `<script>`)

The script is wrapped in an IIFE with `'use strict'`. Everything is local to it; nothing is exported to `window`.

- **DOM references** are grabbed once at the top (`canvas`, HUD elements, overlay, touch controls, buttons).
- **Persistence** uses `localStorage` with `skyfighter.*` keys (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should tolerate missing or blocked storage.
- **Config tables**:
  - `DIFFICULTIES` — spawn rates, enemy speed/shoot chance, starting lives and bombs, boss HP multiplier for `easy` / `normal` / `hard`.
  - `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
- **Audio**: `ensureAudio()` lazily creates the `AudioContext` (must run from a user gesture). `beep()`, `noiseBurst()`, and named SFX helpers synthesize sound; there are no audio files.
- **Logical resolution**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the viewport. Gameplay coordinates are in base units.
- **Input**: keyboard (`keys[e.code]`; arrows/WASD move, Space fires, X/Shift bomb, P/Esc pause), mouse (steer and click-to-fire via `canvasPointFromEvent`), and touch (`joystickStart/Move/End` on the left zone, FIRE and BOMB buttons).
- **Game state** is plain module-level `let` variables reset by `resetGame()`. Core loop functions:
  - `update(dt)` — movement, collisions (`rectsOverlap`), spawning, scoring, power-ups, stage progression.
  - `updateBoss(dt)` / `killBoss()` — boss phases and attack patterns (`fire*` helpers).
  - `draw()` — renders the frame, including `drawPlane`, `drawBossShip`, `drawClouds`, and screen-shake offsets.
  - `loop(now)` — `requestAnimationFrame` driver. Delta time is clamped to 50 ms, and updates run only when `running && !paused`.
- **Lifecycle**: `loseLife()`, `endGame()`, `togglePause()`. The overlay (`#overlay`) is shown for the title, pause, and game-over states.
- **Accessibility**: `announce()` writes to the `#srLive` aria-live region. Reduced motion is honored from `prefers-reduced-motion` and the in-game toggle, and gates screen shake, hit flashes, and the low-health pulse.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a build step, framework, or external dependencies without asking.
- Use the existing style: `const`/`let`, arrow functions for callbacks, 2-space indentation, single quotes in JS.
- Tune gameplay through the config tables (`DIFFICULTIES`, `BOSSES`, `BOSS_PATTERNS`, `THEMES`) rather than hard-coding magic numbers in update logic.
- Any new persisted setting should use a `skyfighter.`-prefixed key and read/write through a try/catch-safe path.
- Keep the game working on mobile: preserve the touch controls and the `viewport-fit=cover` / `user-scalable=no` meta setup.
- Do not add external network calls, analytics, or third-party assets.

## Making changes

1. Read the relevant section of `index.html` before editing; the file is large, so use targeted searches (e.g. `grep -n "function update"`) instead of reading it whole.
2. Make the smallest change that solves the problem.
3. Test in a real browser: start a game, check the HUD, play through at least one stage transition, verify pause/resume, and confirm no console errors. For boss or power-up changes, test those paths explicitly.
4. Commit with a clear message describing the gameplay or UI effect.

## Git

- Development branch for this work: `claude/gifted-tesla-cy0bvg`. Do not push to other branches without permission.
- Do not open pull requests unless explicitly asked.
