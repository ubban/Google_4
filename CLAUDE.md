# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up, shipped as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (~360 lines), CSS in `<style>`, and JavaScript in one IIFE `<script>` (~1,250 lines). |
| `README.md` | Placeholder title only. |

There is no `package.json`, build step, bundler, test suite, linter, or CI configuration. Do not add tooling unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install step is needed.

## Code layout inside `index.html`

- **Markup**: `<canvas id="gameCanvas">`, the HUD (score, best, multiplier, weapon, hearts, bombs), the overlay card (title, difficulty buttons, stats row, start button), the stage banner, combo pop, vignette and flash layers, touch controls, and a visually hidden `aria-live` region (`#srLive`) used by `announce()`.
- **Constants** (near the top of the script): `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage, cycled), `ENEMY_TYPES`, and `BOSSES`. Tune balance here rather than in the update loop.
- **Persistence**: `localStorage` keys are namespaced `skyfighter.*` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Keep that prefix for any new key.
- **Audio**: a lazily created Web Audio context (`ensureAudio()`), with `beep()` and `noiseBurst()` as the primitives. Audio starts only after a user gesture.
- **Game loop**: `loop(now)` clamps `dt` to 50 ms, calls `update(dt)` then `draw()` while running and not paused. `update` handles spawning, collisions, weapons, bombs, and boss patterns. `draw` handles the parallax clouds, planes, bosses, bullets, and effects.
- **Bosses**: `BOSS_PATTERNS` lists attack functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` cycles through them.
- **Adaptive quality**: `trackPerf` watches a rolling average of frame times and steps a `quality` level (0–2) up or down.
- **Input**: keyboard (arrows/WASD to move, Space to fire, X or Shift for bomb, P or Escape to pause), mouse/pointer on the canvas, and on-screen joystick plus fire and bomb buttons for touch.
- **Accessibility**: respects `prefers-reduced-motion` (overridable with the Motion button) and sets a `reduced-motion` class on `<body>`.

## Conventions

- Keep the game as one file. Do not split it into modules or add a build step unless the user asks.
- Match the existing style: `'use strict'` inside the IIFE, `const` by default, `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for top-level constant tables, and short section comments such as `// ---------- Name ----------`.
- Keep the comment density low, the same as the surrounding code.
- Draw with canvas primitives. Do not add image or audio assets unless the user provides them.
- Any change to rendering or input must still work on touch devices and at phone widths. The canvas scales via `resize()` and `BASE_W` / `BASE_H` (480×720).

## Verifying changes

There is no automated test or lint command. Verify by:

1. Loading `index.html` in a browser and checking the browser console for errors.
2. Playing through start, pause/resume, a life lost, a bomb, a stage clear, and game over.
3. Checking the touch controls with a touch emulator when input code changes.

If a headless browser is available, a short Playwright script that loads the page and fails on any `console` error is a reasonable smoke check. Do not commit that script unless asked.

## Git

- Develop on the branch the session names. Do not push to `main` without explicit permission.
- Write commit messages in the imperative mood, with a short subject line and a body that explains the why for larger changes, following the style of the existing history.
- Do not open pull requests unless the user asks for one.
