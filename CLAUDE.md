# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, single-page HTML5 game project. It currently contains one game, **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`.

## Layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (~line 1–363), CSS (inline `<style>`, ~line 10–362), canvas element, and the game script (inline `<script>`, ~line 422 onward). |
| `README.md` | Placeholder README. |

There is no build system, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install step is needed.

## Architecture of `index.html`

The game script is one IIFE with `'use strict'`. Everything lives inside it; nothing is exported to `window`.

- **Loop:** a single `requestAnimationFrame` loop (`loop(now)`). `dt` is clamped to 0.05 s. Each frame runs `trackPerf(dt)`, then `update(dt)` and `draw()` when the game is running and not paused.
- **State and rendering:** `update(dt)` (~line 1148) advances entities, collisions, and spawns. `draw()` (~line 1520) renders the frame. Keep logic in `update` and drawing in `draw`.
- **Adaptive quality:** `quality` (0 full, 1 medium, 2 low) is driven by a rolling average of frame times in `trackPerf`. Visual effects should respect this level rather than being fixed at full quality.
- **Difficulty:** the `DIFFICULTIES` table (~line 485) sets spawn rates, enemy speed, enemy fire chance, lives, bombs, and a boss HP multiplier. Add tuning there rather than hard-coding numbers in logic.
- **Stages and content:** `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss definitions with an HP multiplier). Boss attacks are in `BOSS_PATTERNS` (~line 1372), built from `fire*` helper functions.
- **Audio:** Web Audio, created lazily by `ensureAudio()`. Sound effects are built from `beep()` and `noiseBurst()`, and the `sfx` object groups them. The audio context is only created on the first user gesture (browsers block audio before one), so `ensureAudio()` must be called from an input handler.
- **Persistence:** `localStorage` keys, all prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access should tolerate storage being unavailable where feasible; the existing code currently reads and writes these directly.
- **Accessibility:** `prefers-reduced-motion` seeds the reduced-motion setting, which the user can override with the motion button. Screen-reader announcements go through `announce()` into the `#srLive` region.

## Controls

- Move: arrow keys or WASD; on touch, a joystick in the left zone.
- Fire: hold Space (or the fire button / mouse press). Mouse movement also moves the ship when active on the canvas.
- Bomb: `X` or either Shift key (or the bomb button).
- Pause: `P` or `Escape`.

Space's default is prevented to avoid page scrolling.

## Conventions

- Keep the game self-contained in `index.html` unless the user asks to split it. Do not introduce external CDN dependencies for the game itself.
- Match the existing style: `const` by default, two-space indentation, arrow functions for callbacks, and section comments written as `// ---------- Name ----------`.
- Keep the layout working at phone width. Touch controls are part of the game, not an afterthought.
- Preserve the `prefers-reduced-motion` and mute behavior when adding new visual or audio effects.
- Changes to tuning values belong in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) where possible.

## Verifying changes

There are no automated tests. For gameplay or rendering changes, load `index.html` in a browser and check:

1. The game starts from the overlay on each difficulty.
2. Weapons, bombs, and power-ups behave as expected.
3. A boss fight runs through to its defeat and the stage advances.
4. Game over updates the best score and it persists after reload.
5. Pause, mute, and motion toggles still work, and the layout holds on a narrow viewport.

Check the browser console for errors on load and during play.

## Git

- Development branch for this work: `claude/gifted-tesla-o18z6r`. Push there with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
