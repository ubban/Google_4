# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one self-contained file, `index.html` (inline `<style>` and one inline `<script>`, drawn on a `<canvas>`). There is no build step, package manager, test suite, or server requirement.

- `index.html` — the whole game (~1,670 lines)
- `README.md` — a one-line title; no setup instructions

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There is no build, lint, or test command. Verify changes by playing the game in a browser, ideally at both desktop and phone-sized viewports, since touch controls are part of the design.
- Keep the game dependency-free: do not add npm packages, CDN scripts, or a bundler unless the user asks for one.

## Code layout inside `index.html`

The script is wrapped in a single IIFE with `'use strict'`. Nothing is exported to `window`. Within it, the code is organized roughly as:

1. **DOM references** — `getElementById` lookups at the top of the script.
2. **Constants and tables** — `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six bosses), `KILLS_PER_STAGE`, `BASE_W`/`BASE_H` (480×720 logical resolution).
3. **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
4. **Audio** — procedural WebAudio (`beep`, `noiseBurst`); no audio files. Audio is created lazily via `ensureAudio()`.
5. **Rendering helpers** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
6. **Entity logic** — `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`.
7. **Boss patterns** — `BOSS_PATTERNS` array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, etc.) driven by `updateBoss(dt)`.
8. **Game state flow** — `resetGame`, `loseLife`, `killBoss`, `endGame`, `togglePause`.
9. **Input** — keyboard listeners, mouse/pointer steering via `canvasPointFromEvent`, and touch joystick/fire/bomb controls.
10. **Main loop** — `loop(now)` runs `trackPerf`, then `update` and `draw` while the game is running and not paused. `dt` is clamped to 50 ms.

Important behaviors to preserve when editing:

- **Adaptive quality**: `trackPerf` steps a `quality` level (0 full, 1 medium, 2 low) based on a rolling average of frame time. Visual effects should respect this level rather than assume full quality.
- **Reduced motion**: a user toggle (stored in `skyfighter.reducedMotion`) defaults to the `prefers-reduced-motion` media query. Screen shake and flashes should honor it.
- **Accessibility**: `announce()` writes to a live region (`srLive`) for important game events. Keep UI state changes reflected in the DOM, not only on the canvas.
- **Canvas scaling**: `resize()` fits the 480×720 logical canvas to the viewport. Coordinates are in logical units; use `canvasPointFromEvent` for pointer input rather than raw client coordinates.

## Controls (keep the in-game hint in sync)

The on-screen hint (`#controlsHint`) is the source of truth for players. If you change a control, update that text too.

- Mouse: move to steer, click to fire
- Arrows / WASD: move; Space: fire
- X or Shift: smart bomb
- P or Esc: pause
- Touch: drag the left zone to fly, tap FIRE, tap BOMB

## Conventions

- Keep the single-file structure unless the user asks to split it.
- Match the existing style: 2-space indent, `const`/`let`, arrow functions for callbacks, short tables for data-driven tuning.
- Tune gameplay through the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in logic.
- Don't add comments that restate the code. Section headers in the `// ---------- Name ----------` style are the convention for grouping.
- Persisted values are read defensively with fallbacks (`|| 0`, `|| 'normal'`). Preserve that pattern when adding new stored settings, and use the `skyfighter.` key prefix.

## Git

- Development happens on the branch designated for the session (currently `claude/gifted-tesla-9f31w5`). Don't push to other branches without explicit permission.
- Don't open pull requests unless the user explicitly asks for one.
