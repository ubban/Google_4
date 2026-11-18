# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, browser-based, Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: markup, CSS (`<style>` block), and JavaScript (`<script>` block, wrapped in an IIFE, `'use strict'`). No build step, no dependencies, no bundler.
- `README.md` — minimal placeholder; currently has no setup or usage docs.

There is no package manifest, test suite, linter, or CI configuration in the repo.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Code layout inside `index.html`

The JavaScript is organized by comment-headed sections. Keep new code in the matching section:

- **DOM references** — `getElementById` lookups at the top of the IIFE.
- **Storage keys** — `STORAGE_*` constants for `localStorage` (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
- **Adaptive performance** — `trackPerf()` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low).
- **Config tables** — `DIFFICULTIES` (spawn rates, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, in stage order).
- **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object. Audio is created lazily on first user gesture.
- **HUD and effects** — hearts/bombs rendering, flashes, combo pop, stage banner, screen shake.
- **Layout** — `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport.
- **Input** — keyboard (`keys` map, `keydown`/`keyup`), mouse/touch canvas steering, a virtual joystick, and FIRE/BOMB touch buttons.
- **Game state** — `resetGame()`, `update(dt)`, `applyPowerUp()`, `useBomb()`, `fireWeapon()`, `loseLife()`, `endGame()`, `togglePause()`.
- **Bosses** — `BOSS_PATTERNS` (bullet-pattern functions such as radial, spiral, cross, homing), `updateBoss()`, `killBoss()`.
- **Rendering** — `drawPlane()`, `drawBossShip()`, `drawClouds()`, and `draw()`.
- **Main loop** — `loop(now)` runs `update` and `draw` via `requestAnimationFrame`, with `dt` clamped to 50 ms.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce a build tool, framework, or npm dependencies without explicit approval.
- **Vanilla JS, ES2015+.** Use `const`/`let`, arrow functions, and template literals to match the existing style. Keep everything inside the existing IIFE; do not add globals.
- **Tune gameplay through the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in update/draw logic.
- **Time-based movement.** Updates take `dt` in seconds; scale per-frame motion by `dt` so behavior is frame-rate independent.
- **Accessibility.** Preserve the `announce()` live-region calls, the reduced-motion toggle (`body.reduced-motion`, which honors `prefers-reduced-motion` unless the user overrides it), and the keyboard controls listed below.
- **Persistence.** Only the four `skyfighter.*` keys are stored in `localStorage`. Any new persisted setting needs a matching `STORAGE_*` constant.
- **Mobile.** Touch support is required: the viewport meta disables zoom, and the joystick and FIRE/BOMB buttons must keep working. Test layout changes at phone width.

## Controls (for reference when changing input code)

- Mouse: move to steer, click to fire.
- Arrow keys or WASD: move. Space: fire. X, Shift: bomb. P or Esc: pause.
- Touch: drag left side to fly (virtual joystick), tap FIRE, tap BOMB.

## Testing

No automated tests exist. Verify changes by loading `index.html` in a browser and playing through at least one stage, a boss fight, and a game-over. For changes touching input or layout, check both desktop and a narrow mobile viewport. Browser automation (Playwright) is available in this environment if a scripted check is useful.

## Git

- Development branch: `claude/gifted-tesla-9yeds6`. Push there; do not push to other branches without permission.
- Do not create pull requests unless explicitly asked.
