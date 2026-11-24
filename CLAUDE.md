# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up
built as an HTML5 canvas app. Everything lives in `index.html`: markup, CSS
(`<style>`, lines 10–359), canvas element, and one IIFE `<script>`
(lines 422–1667) with all game logic. `README.md` is a placeholder.

There is no package manager, build step, bundler, test suite, linter, or CI
configuration. Do not invent commands for these. To run the game, open
`index.html` in a browser (or serve the directory with any static server).

## Repository layout

- `index.html` — the entire game (HTML, CSS, JS).
- `README.md` — placeholder; currently only the project title.
- `CLAUDE.md` — this file.

## Architecture of `index.html`

The script is wrapped in `(function () { 'use strict'; ... })()`. Nothing is
exported to `window`; keep new code inside that IIFE.

Key sections, in file order:

- **DOM references** — `const` lookups by id at the top of the script. Any new
  HUD or overlay element needs an id in the markup and a lookup here.
- **Storage keys** — `skyfighter.*` in `localStorage` (best score, mute,
  difficulty, reduced motion). Reads are unguarded today; wrap any new
  storage access in try/catch if it could run in a restricted context.
- **Adaptive quality** — `trackPerf` samples frame time and steps a `quality`
  level (0 full, 1 medium, 2 low) to reduce effects on slow devices.
- **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage
  sky palettes), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, `panFor`, and named SFX.
  Audio is created lazily on the first user gesture (`ensureAudio`); keep it
  that way to satisfy browser autoplay rules.
- **HUD helpers** — hearts, bombs, combo pop, stage banner, screen shake, flash.
- **Layout / input** — `resize` scales the `BASE_W × BASE_H` (480×720) logical
  space to the viewport. Keyboard, mouse, and touch (joystick + fire/bomb
  buttons) feed the same `keys` / `firing` / `mouseActive` state.
- **Game state** — `resetGame`, `player`, and the entity arrays (bullets,
  enemies, boss, power-ups, explosions).
- **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw`.
- **Simulation** — `update(dt)` runs spawns, collisions, scoring, and power-ups.
  Boss attacks are data-driven: `BOSS_PATTERNS` entries call the `fire*`
  helpers (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Flow** — `loseLife`, `endGame`, `togglePause`, and the `loop(now)` RAF
  driver. `dt` is clamped to 50 ms so a backgrounded tab cannot cause a jump.

## Controls (for reference when editing input code)

- Move: arrow keys or WASD; mouse/touch drag on the canvas.
- Fire: Space (hold), or hold the mouse button / fire button.
- Bomb: `X` or Shift (limited per difficulty; clears the screen and damages bosses).
- Pause: `P` or Escape.

## Conventions

- Plain ES2015+ JavaScript and CSS in one file. No frameworks, no modules, no
  external dependencies. Do not add a CDN script or a build step without an
  explicit request.
- Keep the existing style: two-space indent, `const` by default, arrow
  functions for callbacks, section comments with `// ---------- Name ----------`.
- Keep gameplay constants in the tuning tables rather than scattering magic
  numbers through update/draw code.
- Respect the reduced-motion setting: effects that shake or flash should go
  through the existing `reducedMotion` checks.
- Preserve touch support. Any new control needs a touch path as well as a
  keyboard path, and touch handlers use `{ passive: false }` with
  `preventDefault` where needed.
- Keep the game playable on phone-width viewports; the canvas scales via `resize`.

## Verifying changes

Because there is no automated test setup, verify by running the game:

1. Open `index.html` in a browser, or serve the folder statically.
2. Exercise the path you changed: start, fire, bomb, pause, lose a life, reach a
   stage boss, and game over.
3. Check the browser console for errors. Changes to `update`/`draw` should not
   produce exceptions across several stages.
4. For layout changes, check both desktop and a phone-sized viewport.

If you add tooling (a test runner, linter, or build), document its commands
here in the same change.

## Git and delivery

- Default working branch for automated sessions: `claude/gifted-tesla-14j2y0`.
- Commit messages: imperative subject line, a short body explaining the why.
- Do not open pull requests unless explicitly asked.
