# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup (~lines 1–421), CSS (`<style>`, lines 10–359), and one inline `<script>` IIFE (lines 422–1667) containing all game logic, rendering, audio, and input.
- `README.md` — minimal placeholder; not a source of behavior.

There is no build step, bundler, package manager, framework, or test suite. Do not introduce one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by loading the page and playing: check the start overlay, a full stage, the stage boss, pause/resume, game over, and the best-score update.
- For mobile-layout changes, check at phone width; the touch controls (joystick, FIRE, BOMB) only appear on touch-capable devices.

## Architecture notes

Key sections inside the inline script, in roughly this order:

- **DOM refs and storage keys** — element lookups at the top; persisted settings use `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). All storage access should tolerate being unavailable.
- **Adaptive quality** — `trackPerf` samples frame time and steps `quality` (0 full / 1 medium / 2 low) to shed visual cost on slow devices.
- **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Balance changes usually belong here rather than in the logic.
- **Audio** — Web Audio synthesized sound effects (`beep`, `noiseBurst`, `sfx.*`). Audio is created lazily via `ensureAudio()` after a user gesture; respect the mute setting.
- **Input** — keyboard (arrows/WASD move, Space fire, X/Shift bomb, P/Esc pause), mouse on the canvas, and touch joystick plus on-screen buttons.
- **Game state** — `resetGame()` initializes the run; `update(dt)` advances the simulation each frame; `applyPowerUp`, `fireWeapon`, `useBomb`, and the `fire*` boss pattern helpers implement combat.
- **Rendering** — `drawPlane`, `drawBossShip`, and the draw routines for clouds, bullets, power-ups, and effects. Reduced-motion and quality level gate effects such as screen shake and particles.
- **Main loop and pause** — `loop(now)` runs `trackPerf`, then calls `update` and `draw` only when `running && !paused`. `togglePause()` controls the pause overlay.

## Conventions

- Keep the game in a single `index.html` unless the user asks for a restructure. Keep the code inside the existing IIFE and `'use strict'`; avoid leaking globals.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, and short comment banners (`// ---------- Section ----------`) to delimit sections.
- Use the existing `announce()` helper for screen-reader updates, and keep ARIA labels and `title` attributes on controls.
- Preserve accessibility and responsiveness: the canvas scales via `resize()` with a fixed logical size (`BASE_W` × `BASE_H` = 480 × 720). Keep the layout usable on phones without horizontal scroll.
- Do not add external CDN scripts or fonts without a clear reason; the game is intentionally self-contained.

## Git workflow

- Develop on the branch `claude/gifted-tesla-o4yrgs`. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user explicitly asks for one.
- Commit only when asked, or when the task calls for it. Keep commit messages descriptive, and follow the attribution lines supplied by the session.
