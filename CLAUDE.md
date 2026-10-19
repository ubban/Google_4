# CLAUDE.md

Guidance for working in this repository.

## What this repo is

A small, dependency-free browser game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up). The whole game is one file, `index.html`, which contains the markup, CSS, and JavaScript inline. `README.md` is a placeholder with no project docs.

There is no build step, package manager, test suite, linter, or server code. Open `index.html` directly in a browser to run it.

## Files

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: `<style>` (~lines 10–359), HTML body (~361–421), and one IIFE `<script>` (~422–1667). |
| `README.md` | Placeholder title only. |
| `CLAUDE.md` | This file. |

## Running and verifying

- Run: open `index.html` in a browser (or serve the directory with any static server, e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, checking the console for errors, and exercising both keyboard and touch input when the change touches input or layout.
- Syntax-check the JavaScript after edits if convenient, for example by extracting the inline `<script>` and running `node --check` on it.

## Architecture of `index.html`

Everything runs inside a single `(function () { 'use strict'; ... })();` IIFE, so nothing leaks to globals. Keep new code inside it.

Major sections, in the order they appear in the script:

1. **DOM refs and constants**: element lookups (`const xEl = document.getElementById(...)`), `STORAGE_*` keys, and `announce()` for screen-reader live-region updates.
2. **Adaptive performance** (`trackPerf`): a `quality` level (0 full, 1 medium, 2 low) is nudged up or down from a rolling average of frame times.
3. **Tuning data**:
   - `DIFFICULTIES`: spawn rate, enemy speed and shoot chance, lives, bombs, and boss HP multiplier per difficulty (`easy`, `normal`, `hard`).
   - `KILLS_PER_STAGE` (18): kills needed before a boss warning.
   - `THEMES`: sky gradient, cloud, and sun colors per stage, cycled through.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (4 HP, minimum stage 2).
   - `BOSSES`: six bosses (VANGUARD through TITAN), each with colors and an `hpMul`.
   - `BOSS_PATTERNS` (~line 1372): bullet-pattern functions such as radial, aimed spread, spiral, cross, homing, and double ring.
4. **Audio**: `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture first). `beep()` and `noiseBurst()` synthesize all sound effects; there are no audio files. `muted` is persisted.
5. **UI helpers**: hearts and bombs rendering, hit and bomb flashes, combo pop, stage banner, screen shake, and mute and motion toggles.
6. **Layout and input**:
   - `resize()` scales the fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport.
   - Keyboard (`keys` map keyed by `e.code`): arrows or WASD to move, Space to fire, `X` or Shift for a bomb, `P` or Escape to pause.
   - Touch: a virtual joystick (`moveZone`), a fire button, and a bomb button, shown when `isTouchDevice` is true.
   - Pointer input is translated with `canvasPointFromEvent()`.
7. **Game state and logic**:
   - `resetGame()` initializes state.
   - `update(dt)` runs the simulation: movement, firing, enemy spawning, collisions (`rectsOverlap`), power-ups, boss logic, and score and combo.
   - `loseLife()`, `killBoss()`, `endGame()`, and `togglePause()` handle state transitions.
8. **Rendering**: `draw()` renders the frame with the 2D canvas API. Helpers include `drawPlane`, `drawBossShip`, `drawClouds`, and `spawnExplosion`.
9. **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw` while the game is running and not paused. It is driven by `requestAnimationFrame`.

### State and persistence

Persisted values use `localStorage` under the `skyfighter.*` prefix:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every `localStorage` read and write is currently unguarded. If you add more, consider wrapping them in `try/catch`, since storage can be unavailable in private windows or with blocked site data.

### Accessibility

- `prefers-reduced-motion` is honored by default and can be overridden with the motion toggle, which also adds a `reduced-motion` class to `<body>`.
- Status changes go through `announce()` into the `#srLive` `aria-live` region. Use it for new announcements rather than adding ad hoc live regions.

## Conventions

- Keep to the existing style: 2-space indentation, single quotes in JS, `const` and `let` (no `var`), arrow functions for callbacks, and section headers written as `// ---------- Name ----------`.
- Add tuning values to the existing data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) instead of hard-coding magic numbers in logic.
- Element IDs and CSS class names are referenced both from markup and from script, so rename both sides together.
- Do not add external dependencies, CDN scripts, or a build step unless the user asks for it. The game is intentionally a single self-contained file.
- Respect the reduced-motion setting and the touch controls when adding visual effects or new input.

## Git

- Develop on the branch assigned for the session; do not push to other branches without explicit permission.
- Keep commits focused, with descriptive messages in the imperative mood.
