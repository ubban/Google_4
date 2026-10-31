# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up with weapons, smart bombs, stage waves, and bosses.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines).
- `README.md` — minimal placeholder.

There is no build step, no package manager, no dependencies, and no test suite.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- Verify changes by playing them in a browser. There are no automated tests or linters configured.

## Code layout (inside `index.html`)

The game script is one IIFE (`(function () { 'use strict'; ... })();`) inside the `<script>` tag. Key sections, in file order:

- **DOM references** — `getElementById` lookups for HUD, overlay, touch controls, and boss bar.
- **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all prefixed `skyfighter.`).
- **Adaptive quality** — `trackPerf()` steps `quality` (0 full / 1 medium / 2 low) based on a rolling frame-time average.
- **Config tables** — `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
- **Audio** — Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on first user gesture.
- **HUD and feedback** — hearts, bombs, combo pop, stage banner, screen shake, hit flash, announcements to a `aria-live` region.
- **Input** — keyboard (`keys[e.code]`), mouse steering and click-to-fire, a virtual joystick and FIRE/BOMB buttons for touch.
- **Game state** — `resetGame()`, `update(dt)`, `fireWeapon()`, `spawnEnemy()`, `spawnBoss()`, `applyPowerUp()`, `useBomb()`, `loseLife()`, `endGame()`, `togglePause()`.
- **Boss patterns** — `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring) used by `updateBoss()`.
- **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`), drawn with the 2D canvas API.
- **Main loop** — `loop(now)` drives `update` and `draw` with `requestAnimationFrame`.

## Controls (keep the in-game hint in sync)

The control hint lives in `#controlsHint` in the overlay markup. If you change a binding, update that text too.

- Move: Arrow keys or WASD; mouse steers when no key is held.
- Fire: Space (or hold the mouse button / FIRE button).
- Bomb: X or Shift (or BOMB button).
- Pause: P or Escape.
- Touch: drag the left zone to fly, tap FIRE, tap BOMB.

## Persistence

State is stored in `localStorage` under these keys. Reads are guarded only by fallbacks (`|| 0`, `|| 'normal'`), so keep that pattern for any new key:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; defaults to the `prefers-reduced-motion` media query when unset |

Wrap new `localStorage` access in `try/catch` if it could run where storage is blocked (private windows, restricted iframes).

## Conventions

- **Keep it a single file.** Do not introduce a build step, framework, bundler, or external runtime dependency unless the user asks for it.
- **Match the existing style:** 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, section headers written as `// ---------- Name ----------`.
- **Preserve the IIFE and `'use strict'`.** Nothing should leak to `window` except the intended globals.
- **Time-based updates.** Movement and timers use `dt` (seconds, from `loop`). Don't tie speeds to frame count.
- **Respect reduced motion.** Effects such as screen shake, flashes, and parallax should check `reducedMotion` / the `reduced-motion` body class.
- **Keep the layout phone-friendly.** The canvas scales via `resize()` with a fixed `BASE_W × BASE_H` (480 × 720) logical size; the viewport meta disables user scaling, so test touch layouts.
- **Accessibility.** Status text goes through `announce()` so screen readers hear it.

## Git

- Development branch for this work: `claude/gifted-tesla-mmrphh`. Default branch: `main`.
- Write clear, imperative commit messages describing the gameplay or code change.
