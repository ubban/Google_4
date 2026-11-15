# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file:

- `index.html` (~1,670 lines): inline `<style>`, markup, and one inline `<script>` wrapped in an IIFE. There are no external scripts, stylesheets, or fonts (the only link is an inline SVG favicon).
- `README.md`: a placeholder. It is not a source of project documentation.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Nothing needs to be installed.

## Running and testing

- Run the game by opening `index.html` directly in a browser, or by serving the directory with any static server (for example `python3 -m http.server` from the repo root).
- There are no automated tests. Verify changes by playing the game in a browser, ideally at both desktop and phone-sized viewports, since the game has touch controls.
- Syntax-check the script after edits, for example by extracting the inline `<script>` and running `node --check` on it.

## Code layout inside `index.html`

The inline script is organized into commented sections. Keep new code in the matching section:

- **Setup and DOM refs**: `getElementById` lookups at the top of the IIFE.
- **Storage keys**: `STORAGE_*` constants for `localStorage` (best score, mute, difficulty, reduced motion). Keys are namespaced `skyfighter.*`.
- **Adaptive performance** (`trackPerf`): adjusts a `quality` level (0 full, 1 medium, 2 low) from average frame time over 45 frames.
- **Tuning data**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES`. Balance changes should usually go here rather than into logic.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is created lazily on the first user gesture and respects the mute setting.
- **Input**: keyboard (`keys` map keyed by `KeyboardEvent.code`), touch joystick and buttons, and mouse via the canvas.
- **Game state**: `resetGame`, `update(dt)` for simulation, `draw()` for rendering, and `loop(now)` as the `requestAnimationFrame` driver.
- **Power-ups, weapons, bombs, bosses**: `applyPowerUp`, `fireWeapon`, `useBomb`, and the `BOSS_PATTERNS` bullet-pattern functions.

Time-based logic uses `dt` in seconds, clamped to 0.05 in `loop`. Preserve that clamp; without it, a backgrounded tab causes large jumps.

## Controls (for manual testing)

- Move: arrow keys or WASD.
- Fire: Space (hold), or the fire button on touch devices.
- Bomb: X or either Shift key, or the bomb button.
- Pause: P or Escape.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add a build system, framework, or dependencies without asking.
- Use plain JavaScript (ES2015+) with `const` and `let`, matching the existing style: two-space indentation, single quotes, and arrow functions for callbacks.
- Respect `reducedMotion`: new screen shake, flashes, or particle-heavy effects should check the flag and the current `quality` level.
- Keep the accessibility hooks working. Status messages go through `announce()`, which writes to the `srLive` live region. Use it for important game events rather than adding new live regions.
- Keep touch and keyboard input paths in parity when adding new actions.
- Persist settings only through the existing `STORAGE_*` keys, and wrap new `localStorage` access in a way that still works if storage is unavailable (private mode, blocked site data).

## Git

- Work on the branch assigned for the session. Do not push to other branches without explicit permission.
- Commit messages should be short and imperative, describing the gameplay or code change (e.g. "Add boss telegraph before pattern fire").
