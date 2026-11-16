# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-em-up.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), canvas element, and one inline `<script>` IIFE (~lines 422–1668) containing all game logic, rendering, audio, and input.
- `README.md` — minimal title only.

There is no build step, package manager, test suite, or linter. Nothing to install.

## Running it

Open `index.html` directly in a browser, or serve the directory statically (e.g. `python3 -m http.server`) and visit it. Verify changes by playing in a browser; there are no automated tests to run.

## Architecture

Everything lives in one IIFE inside `index.html`:

- **Config constants** (top of the script): `DIFFICULTIES` (easy/normal/hard tuning for spawn rate, enemy speed, shot chance, lives, bombs, boss HP), `KILLS_PER_STAGE`, `THEMES` (sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss definitions), `BOSS_PATTERNS` (bullet patterns).
- **Canvas sizing**: logical resolution `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()`.
- **Game loop**: `loop(now)` drives `update(dt)` then `draw()` each frame via `requestAnimationFrame`. `dt` is clamped to 50 ms. `trackPerf()` adapts a `quality` level (0–2) from frame timings.
- **Update/draw**: `update()` handles movement, firing, collisions, power-ups, and stage progression; `updateBoss()` / `killBoss()` handle boss fights; `draw()` renders everything including `drawPlane`, `drawBossShip`, and clouds.
- **Audio**: Web Audio helpers `beep()` and `noiseBurst()` (no audio files). `ensureAudio()` must run from a user gesture. Mute state is persisted.
- **Input**: keyboard (`keys[e.code]`), mouse/pointer via `canvasPointFromEvent`, and touch (joystick, fire, bomb buttons) shown when `isTouchDevice`.
- **HUD/UI**: DOM elements (hearts, bombs, score, weapon, boss bar, overlays, stage banner, combo pop) are updated from script; screen-reader text goes through `announce()`.
- **Persistence**: `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access should tolerate missing storage gracefully (the current code reads it directly; keep new reads defensive).

## Controls (for manual testing)

- Move: arrow keys / WASD (keyboard) or drag the on-screen joystick (touch)
- Fire: Space (hold); fire button on touch
- Bomb: X or Shift; bomb button on touch
- Pause: P or Escape

## Conventions

- Keep the game as a single self-contained HTML file with no external dependencies unless the user asks otherwise.
- Keep new code inside the existing IIFE and follow its style: `const`/`let`, 2-space indent, single quotes, named constants for tuning values in the config block rather than magic numbers.
- Put new tuning values in the existing `DIFFICULTIES`/`ENEMY_TYPES`/`BOSSES` tables where they fit.
- Preserve accessibility hooks (`announce()`, reduced-motion handling, `aria` attributes) when changing UI.
- Preserve the `prefers-reduced-motion` behavior and the mobile touch controls when changing input or visuals.

## Git

- Development branch for this work: `claude/gifted-tesla-p7ug2x`. Commit and push there; do not push to other branches without permission.
- Keep commit messages descriptive and focused on the gameplay or UI change.
