# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small, dependency-free browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html`:

- `<style>` (approx. lines 10–359): all CSS, including HUD, overlay, touch controls, and responsive layout.
- `<body>` (approx. lines 361–421): the DOM — canvas `#gameCanvas`, HUD elements, boss bar, touch controls, and the start/pause overlay.
- `<script>` (approx. lines 422–1667): one IIFE (`'use strict'`) containing all game logic, rendering, audio, and input.

`README.md` is a placeholder. There is no build step, package manager, test suite, or linter.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, and check both desktop (keyboard/mouse) and touch layouts when touching input or CSS.
- Do not introduce a build step, bundler, or npm dependencies unless the user asks for one. Keep the project as a single self-contained file.

## Code structure (inside the `<script>` block)

- **Config tables** near the top: `DIFFICULTIES` (easy/normal/hard — spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses with colors and HP multipliers). Tune balance here rather than in logic.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Always wrap access in a way that tolerates storage being unavailable if you add new reads/writes, and reuse the existing `STORAGE_*` constants for new keys.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`) with synthesized sound via `beep` and `noiseBurst`. Audio is created only after a user gesture.
- **Accessibility**: `prefers-reduced-motion` is honored by default and can be toggled (`motionBtn`); `announce()` writes to the `aria-live` region `#srLive`. Keep these working when adding effects like screen shake or flashes.
- **Game loop**: `loop(now)` computes `dt` (capped at 0.05 s) and calls `update(dt)` then `draw()`, only while `running && !paused`.
- **Update/draw split**: `update` handles movement, firing, collisions, spawning, and power-ups; `updateBoss` and the `fire*` pattern helpers drive boss attacks (`BOSS_PATTERNS`); `draw` renders everything to the canvas, with helpers `drawPlane`, `drawBossShip`, and `drawClouds`.
- **Sizing**: the canvas uses a fixed logical resolution `BASE_W × BASE_H` (480 × 720), scaled to the viewport by `resize()`. Pointer coordinates are converted with `canvasPointFromEvent`.

## Controls (keep the hint in sync)

- Keyboard: Arrows / WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- Mouse: the plane glides toward the pointer; click to fire.
- Touch: drag left zone to fly (joystick), FIRE and BOMB buttons on the right.

The in-game hint is `#controlsHint` in the overlay. If you change or add a control, update that text too.

## Conventions

- Keep the single-file structure and plain JavaScript (ES2015+ is fine, matching the existing code). Don't split into modules without being asked.
- Use `const` by default and follow the existing naming: `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for config constants.
- Prefer editing the existing data tables and helpers over duplicating logic.
- Keep DOM lookups at the top of the IIFE, as done today.
- Don't add comments that restate the code; the existing section comments (`// ---------- Input ----------`) are the house style for grouping.

## Git

- Develop on the branch specified by the task or session; don't push to other branches without explicit permission.
- Write clear commit messages describing the gameplay or code change.
