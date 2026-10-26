# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up, implemented as a single self-contained file:

- `index.html` (~1,670 lines): all markup, CSS (`<style>`, starts ~line 10), and game logic (one IIFE in an inline `<script>`, starts ~line 422).
- `README.md`: a placeholder title only.

There is no build step, package manager, bundler, test suite, linter, or CI config. Open `index.html` in a browser to run the game.

## Working conventions

- Keep the game in the single `index.html` unless the user asks for a split. Do not introduce a build tool or framework without asking.
- Preserve the IIFE wrapper and `'use strict'`. Game code reads DOM elements once at the top of the script; add new element lookups alongside them rather than querying inside the loop.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, section headers written as `// ---------- Name ----------`, and short comments only where the intent is not obvious.
- Game tuning lives in data tables near the top of the script. Prefer editing those over hard-coding numbers in logic:
  - `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier)
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`)
  - `BOSSES` (six bosses, cycled by stage) and `BOSS_PATTERNS` (attack patterns, ~line 1372)
  - `THEMES` (six background palettes, cycled per stage)
  - `KILLS_PER_STAGE` (18)
- The logical play field is fixed at `BASE_W = 480` by `BASE_H = 720`. `resize()` scales the canvas to fit the window (capped at 1.6x). Coordinates in game logic use the base size, not the CSS size.
- The frame loop (`loop`, ~line 1640) clamps `dt` to 50 ms and runs `update(dt)` then `draw()` only when the game is running and not paused. `trackPerf` adapts a `quality` level (0–2) from average frame time; respect it when adding expensive draw effects.

## Persistence and accessibility

- `localStorage` keys (all prefixed `skyfighter.`): `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads are not wrapped in try/catch today; if you add writes, keep them defensive, since storage can be blocked.
- Reduced motion defaults to the `prefers-reduced-motion` media query and can be overridden with the in-game toggle. Effects should check the `reducedMotion` flag: `screenShake` caps duration at 0.1 s when it is set, and `flashHit` and `flashBomb` return early.
- Use `announce(text)` to update the `#srLive` aria-live region for important state changes (stage, game over, pause).
- Touch controls (`#touchControls`) appear only on touch-capable devices. Any new control needs both keyboard and touch paths.

## Controls (for reference when testing)

- Move: arrow keys or WASD; touch uses the on-screen joystick.
- Fire: hold Space; touch uses the fire button.
- Bomb: X or Shift; touch uses the bomb button.
- Pause: P or Escape.

## Testing and verification

There are no automated tests. To verify a change:

1. Open `index.html` in a browser (or serve the directory with any static server).
2. Play through at least one stage on each difficulty you touched, and check the boss fight if you changed boss or pattern code.
3. Check the mobile layout and touch controls with device emulation if you changed layout or input.
4. Check the browser console for errors.

If you add tests, describe how to run them here.

## Git

- Development happens on feature branches named `claude/...`. Do not push to other branches without explicit permission.
- Write clear commit messages that describe the behavior change, not just the file touched.
