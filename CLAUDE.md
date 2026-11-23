# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser. It is a single self-contained file:

- `index.html` — all markup (`<body>`), CSS (`<style>`, ~lines 10–360), and game logic (one inline `<script>` IIFE, ~lines 422–1668). There is no build step, no bundler, no package manager, and no external dependencies.
- `README.md` — minimal placeholder.

Open `index.html` directly in a browser to play. Nothing needs to be installed.

## Working conventions

- **Keep it dependency-free.** Do not add frameworks, npm packages, or CDN scripts unless the user asks. The game is intentionally a single file that works offline.
- **Stay in `index.html` unless there is a clear reason to split.** If you do split out code, keep the page loadable from `file://` with no server.
- **No build or lint tooling exists.** Verify changes by loading the page in a browser (or a headless Chromium) and playing through the affected code path. Check the browser console for errors.
- **Match the existing style:** 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for callbacks, and the IIFE wrapper with `'use strict'`.
- **Don't add comments that restate the code.** Comment only non-obvious intent.

## Code map (`index.html` script)

Constants and tuning live near the top of the IIFE:

- `DIFFICULTIES` — per-difficulty `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`. Tune difficulty here, not in the update loop.
- `KILLS_PER_STAGE` — kills needed to advance a stage.
- `THEMES` — background palettes cycled per stage (sky gradient, cloud and sun colors, display name).
- `ENEMY_TYPES` and `BOSSES` — enemy and boss definitions (colors, `hpMul`).
- `BOSS_PATTERNS` — boss bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).

Core functions:

- `update(dt)` — per-frame simulation: movement, firing, collisions, spawning, scoring.
- `draw()` — per-frame rendering to the canvas, including `drawPlane`, `drawBossShip`, and `drawClouds`.
- `loop(now)` — `requestAnimationFrame` driver. `dt` is clamped to 0.05 s, so a backgrounded tab does not produce a huge step.
- `resetGame()` — clears state for a new run. Any new per-run state must be reset here.
- `endGame()`, `loseLife()`, `killBoss()`, `applyPowerUp()`, `useBomb()`, `fireWeapon()` — state transitions.
- `resize()` — scales the fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport.

Audio is synthesized with the Web Audio API (`beep`, `noiseBurst`, the `sfx` table). `ensureAudio()` must run from a user gesture before sound plays.

## Persistence

Player settings and the high score are stored in `localStorage` under these keys:

| Key | Purpose |
| --- | --- |
| `skyfighter.bestScore` | Best score |
| `skyfighter.muted` | Mute toggle (`'1'` / `'0'`) |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | Reduced-motion toggle (`'1'` / `'0'`) |

Reads are currently unguarded. If you touch that code, consider wrapping storage access in try/catch, since `localStorage` can throw in some privacy modes.

## Controls (keep the on-screen hint in sync)

The hint text lives in `#controlsHint` (around line 417). If you change controls, update it.

- Keyboard: arrows/WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag the left zone (virtual joystick), tap FIRE, tap BOMB.

Input handlers are registered near lines 769–861 and 1505. Space's default is suppressed on `keydown` to stop page scrolling.

## Accessibility and UX

- The page has a `srLive` live region used by `announce()` for screen-reader updates. Use `announce()` for important state changes.
- A reduced-motion setting exists (`reducedMotion`) and gates screen shake and similar effects. New motion-heavy effects should respect it.
- Touch controls use `touch-action: none` and are shown via `#touchControls.enabled`. Don't break mobile layout when editing CSS.

## Git and branches

- Development branch: `claude/gifted-tesla-tlaiic`. Commit and push there.
- Do not open pull requests unless explicitly asked.
- Commit messages describe the change in the imperative mood, in one short subject line.
