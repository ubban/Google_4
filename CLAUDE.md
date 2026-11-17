# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

Files:

- `index.html` — the entire game: markup, CSS (`<style>` block), and JavaScript (one IIFE in a `<script>` block). About 1,670 lines.
- `README.md` — a stub (title only).

There is no build system, package manager, bundler, test suite, or linter. Do not invent one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

Verify changes by playing them in a browser. There are no automated tests to run.

## Code layout inside `index.html`

- **Markup/CSS**: HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar), overlay/start screen, touch controls (joystick, fire, bomb), and the `#gameCanvas` element. Styles are in the `<style>` block above the canvas.
- **Script structure**: everything lives in a single `(function () { 'use strict'; ... })()` closure. Nothing is exposed on `window`; keep it that way unless there is a reason.
- **Config constants** (top of the script, near the DOM lookups):
  - `DIFFICULTIES` — per-difficulty spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier.
  - `KILLS_PER_STAGE`, `THEMES` (background palettes per stage), `ENEMY_TYPES`, `BOSSES`.
  - `BOSS_PATTERNS` — bullet patterns (radial, spread, spiral, cross, homing, etc.) used by bosses.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads must tolerate missing or invalid values.
- **Game loop**: `loop(now)` calls `trackPerf`, then `update(dt)` and `draw()` while running and not paused. `dt` is clamped to 50 ms.
- **Adaptive quality**: `trackPerf` samples frame times and steps a `quality` level (0–2) up or down. Drawing code should respect it.
- **Audio**: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on the first user gesture (`ensureAudio()` is called from start/fire/bomb handlers); do not create an `AudioContext` at load time.
- **Input**: keyboard (`keys[e.code]`), touch joystick and buttons, and mouse/pointer on the canvas. Keys: arrows/WASD move, Space or the fire button fires, X/Shift uses a bomb, P/Escape pauses.
- **Scaling**: the canvas uses a fixed logical size (`BASE_W = 480`, `BASE_H = 720`) and is scaled in `resize()`. Game coordinates stay in logical units.
- **Accessibility**: an `aria-live` region (`srLive`) is updated via `announce()`. Respect `prefers-reduced-motion` (`reducedMotion`) for screen shake, flashes, and similar effects.

## Conventions

- Keep the single-file structure. Do not split into modules or add a bundler unless the user asks.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, semicolons, single quotes in JS.
- Prefer small, local edits. Reuse the existing helpers (`spawnExplosion`, `addScore`, `screenShake`, `announce`, etc.) instead of duplicating logic.
- Gameplay tuning belongs in the config constants at the top of the script, not scattered through update/draw code.
- Any new user-facing text should be announced through the live region where it is important, and any new motion effect should honor `reducedMotion`.
- Do not add external dependencies, CDN scripts, or network calls to the game.

## Git and branches

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-z7yhem`). Do not push to other branches without explicit permission.
- Do not open pull requests unless the user explicitly asks for one.
- Commit messages should be short and imperative, describing the gameplay or code change.
