# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, dependency-free HTML5 canvas
vertical shoot-'em-up (Raiden-style) game.

- `index.html` — the entire game: inline `<style>`, markup/HUD/overlays, and one
  inline `<script>` wrapped in an IIFE. There is no build step, bundler, package
  manager, or test suite.
- `README.md` — minimal project readme.

To run it, open `index.html` in a browser (or serve the directory with any
static server). Nothing needs to be installed.

## Architecture (inside `index.html`)

The game script is organized as one closure with these sections, roughly in order:

1. **DOM references** — `getElementById` lookups for canvas, HUD, overlays, and
   touch controls. Keep new element lookups grouped here.
2. **Constants** — `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE`,
   `THEMES` (per-stage palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
3. **State and persistence** — best score, mute, difficulty, and reduced-motion
   preferences are read from and written to `localStorage` under `skyfighter.*` keys.
4. **Audio** — a lazily created Web Audio context (`ensureAudio`) plus `beep` and
   `noiseBurst` helpers. Audio is only started after a user gesture.
5. **HUD helpers** — hearts, bombs, combo pops, stage banner, screen shake, hit flash.
6. **Input** — keyboard (`keys` map keyed by `e.code`), pointer/mouse on the canvas,
   and touch controls (virtual joystick, fire, and bomb buttons) shown on touch devices.
7. **Game lifecycle** — `resetGame`, `update(dt)`, `draw()`, `endGame`, `loseLife`,
   `togglePause`. The main loop is a `requestAnimationFrame` loop with `dt` clamped
   to 50 ms.
8. **Gameplay systems** — enemy and boss spawning, weapons (`fireWeapon`, Vulcan/Laser
   with 3 power levels), smart bombs (`useBomb`), power-ups (`applyPowerUp`), kill-chain
   score multiplier (`addScore`, `breakChain`), and boss bullet patterns.
9. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, and the full `draw()`
   pass. Rendering uses the 2D context directly; no sprites or image assets.

### Controls

- Keyboard: arrows/WASD to move, Space to fire, `X` / Shift for a bomb, `P` / Escape to pause.
- Touch: on-screen joystick, fire button, and bomb button.

### Gameplay tuning

- Difficulty presets (`easy`, `normal`, `hard`) set spawn rate, enemy speed, enemy
  fire chance, starting lives, starting bombs, and boss HP multiplier.
- A stage ends after `KILLS_PER_STAGE` kills and is followed by a boss fight.

## Conventions

- **Keep it a single file.** Do not introduce a build step, frameworks, or external
  dependencies unless the user explicitly asks. Styles stay in the `<style>` block and
  logic stays in the inline script.
- **Stay in the IIFE.** Do not add new globals; expose nothing on `window` except
  event listeners.
- **Use `const`/`let`, `'use strict'` is already on.** Match the existing 2-space
  indentation and single-quoted string style.
- **Time-based movement.** Multiply per-frame motion by `dt` so behavior is
  frame-rate independent. Respect the `dt` clamp.
- **Respect reduced motion.** Screen shake, flashes, and similar effects should honor
  the `reducedMotion` setting.
- **Accessibility.** Announcements for screen readers go through the existing
  `announce()` helper (`srLive` live region). Keep HUD changes accessible.
- **Mobile.** The layout must work at phone width with touch controls; the canvas is
  scaled by `resize()` from a `480×720` base size. Don't hard-code pixel positions
  that break at other aspect ratios.
- **Persistence keys** use the `skyfighter.` prefix. Wrap new `localStorage` access so
  the game still runs if storage is unavailable (private mode, blocked storage).

## Testing and verification

There is no automated test suite. Verify changes by:

1. Opening `index.html` in a browser and playing through at least one stage and boss.
2. Checking all three difficulties, pause/resume, mute, and reduced-motion toggles.
3. Checking touch controls with a mobile viewport in dev tools.
4. Confirming the best score persists across a reload.
5. Watching the browser console for errors while playing.

If you add logic that is easy to isolate (e.g. scoring, boss pattern math), consider
exposing it for manual checks rather than adding a test framework.

## Git and PR workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-aqtzm1`).
- Write clear, imperative commit messages that describe the gameplay or UI change.
- Do not open a pull request unless explicitly asked.
