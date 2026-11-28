# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up with weapon switching, smart bombs, stages, and bosses.

The entire game lives in a single file, `index.html` (~1,670 lines): inline `<style>`, markup, and one inline `<script>`. `README.md` is a one-line title.

There is no build system, package manager, test suite, or linter configured.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` and visit `/index.html`).
- **Verify changes:** there are no automated tests. Load the game in a browser and play through the affected behavior. Check the browser console for errors.
- Do not introduce a build step, framework, or npm dependency unless the user asks for it. The project is intentionally a single self-contained file.

## Code layout inside `index.html`

- **Markup and CSS:** the HUD, overlay (start/pause/game-over screens), touch controls, and boss bar are plain DOM elements styled in the inline `<style>` block. Color and layout tweaks belong there.
- **Script:** one IIFE-style block that begins around line 426 and holds all game logic. Key sections, roughly in file order:
  - DOM element lookups (`canvas`, `overlay`, HUD elements, touch controls).
  - Storage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
  - Tuning tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES`, and `KILLS_PER_STAGE`. Balance changes usually happen here first.
  - Audio: a lazily created `AudioContext` (`ensureAudio`) with `beep`/`noiseBurst` primitives and a named `sfx` table.
  - HUD helpers: hearts, bombs, combo popup, stage banner, screen shake, vignette/flash effects.
  - Layout: a fixed logical canvas of **480×720** (`BASE_W`/`BASE_H`), scaled to fit the window by `resize()` with a max scale of 1.6.
  - Input: keyboard state map, touch joystick and fire/bomb buttons, and mouse/pointer targeting.
  - Performance: `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to reduce effects on slow devices.

## Controls (for reference when testing)

- **Move:** arrow keys / WASD via `keys[e.code]`; on touch, a virtual joystick.
- **Fire:** held fire (touch fire button, mouse button).
- **Bomb:** `X` or either Shift key.
- **Pause:** `P` or `Escape`.
- **Mouse:** pointer position sets the ship's target.

## Conventions

- Keep the game in `index.html` unless the user explicitly asks to split it. If you do split files, keep them as plain static assets with no build step.
- Use the existing `SCREAMING_SNAKE_CASE` for constant tables and `camelCase` for functions and state.
- Persist user settings only through the existing `skyfighter.*` localStorage keys. Wrap new storage reads and writes so the game still works when storage is unavailable (private windows, blocked site data).
- Respect the reduced-motion setting: new screen shake, flashes, or particle effects should go through the existing `reducedMotion` checks.
- Keep the layout usable at phone width (touch controls, `touch-action: none`, safe-area insets already handled in the HUD CSS).
- Preserve accessibility hooks already present: the `srLive` live region is used by `announce()` for screen reader messages. Route new status messages through `announce()`.
- Audio should stay muteable via the `muted` flag and created lazily after a user gesture.

## Git

- Develop on the branch assigned for the session. Do not push to other branches without permission.
- Commit messages should be short, imperative, and describe the gameplay or code change.
