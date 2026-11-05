# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–420), and JavaScript (`<script>`, ~line 422 onward), with no external dependencies.
- `README.md` — minimal project title only.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify changes by actually playing: check the title overlay, a stage's waves, a stage boss, pause (`P`/`Esc`), bomb use, game over, and the mobile touch layout.
- Test at phone width as well as desktop; the canvas scales to the viewport via `resize()`.

## Code structure (inside `index.html`)

The script is wrapped in a single IIFE. Key sections, in source order:

- **Storage keys** (`STORAGE_*`): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Reads are at startup (~line 517).
- **Config tables**: `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage), `BOSS_PATTERNS` (bullet patterns).
- **Audio**: WebAudio synthesis only (`beep`, `noiseBurst`); no audio asset files. `ensureAudio()` must run from a user gesture.
- **Input** (~line 766): keyboard (arrows/WASD move, Space fires, `X`/Shift bomb, `P`/`Esc` pause), touch joystick + fire/bomb buttons, and mouse/pointer on the canvas.
- **Game state**: `resetGame()`, then `update(dt)` per frame, and the draw functions (`drawPlane`, `drawBossShip`, etc.).
- **Progression**: kill-chain multiplier (`addScore`, `breakChain`), stages advance after `KILLS_PER_STAGE` kills, and bosses spawn at stage transitions (`spawnBoss`, `killBoss`).
- **Power-ups**: `maybeSpawnPowerUp` / `applyPowerUp` — weapon level (Vulcan/Laser, 3 levels), bombs, extra life.

## Conventions

- Keep the game as a single file unless the user asks for a split. New constants go in the existing config tables rather than being hard-coded in logic.
- Preserve the existing style: 2-space indent, `const`/`let`, ES2015+ syntax, no transpilation.
- Gameplay values are tuned by feel. When changing a number, say so in the commit message and note the before/after value.
- Respect the reduced-motion setting: any new screen shake, flash, or pulse must go through the existing `reducedMotion` check.
- All new UI text should be announced through `announce()` if it conveys game state, for screen readers.
- Any new persistent setting needs a `STORAGE_*` key and a `try`-safe read, since `localStorage` may be unavailable.
- Keep HUD and overlay changes working at phone width with `env(safe-area-inset-*)` padding.

## Git workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-523xtr`). Do not push to other branches without explicit permission.
- Commit with clear, descriptive messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.
