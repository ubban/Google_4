# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game (markup, CSS, and JavaScript) lives in one file, `index.html`, about 1,670 lines.

Other files:
- `README.md`: a title line and a timestamp. It has no setup or run instructions.

There is no package manager, build step, bundler, test suite, linter, or CI configuration. Do not add tooling unless asked.

## Running the project

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server`. No install step is needed.

## Code layout (`index.html`)

- **Lines ~1–365 (`<style>`)**: all CSS. Layout is a fixed 480×720 logical canvas (`BASE_W`/`BASE_H`) that is scaled to fit the viewport.
- **Lines ~363–420 (markup)**: HUD, overlays, touch controls (move zone, fire, bomb), and the `#srLive` aria-live region.
- **Lines ~422–1667 (`<script>`)**: one IIFE with `'use strict'`. Sections, in order:
  - DOM references and storage keys (`skyfighter.*` in `localStorage`).
  - Adaptive quality (`trackPerf`, `quality` 0–2) that reduces effects when frame times are slow.
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (background palettes per stage), `ENEMY_TYPES`, `BOSSES`.
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is synthesized with WebAudio, with no asset files.
  - Input: keyboard (`keys`), virtual joystick, mouse/pointer on the canvas, and fire/bomb buttons.
  - Game state and logic: `resetGame`, `update(dt)`, weapons (`fireWeapon`), power-ups, bombs (`useBomb`), scoring/combo (`addScore`, `breakChain`), enemy and boss spawning, and `loseLife`/`endGame`/`togglePause`.
  - Boss attack patterns: `BOSS_PATTERNS` and the `fire*` helpers.
  - Rendering: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
  - Main loop: `loop(now)` uses `requestAnimationFrame` and clamps `dt` to 50 ms.

Game logic and drawing are in the same IIFE and share module-level `let` state. When you add a feature, follow the existing pattern: tune values in the tables at the top of the script, and keep update and draw code separate.

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Do not introduce a framework or external dependencies.
- Use plain ES2015+ JavaScript and the Canvas 2D API. Match the existing style: 2-space indent, single quotes, `const` by default.
- Preserve accessibility hooks. The `#srLive` region announces state changes through `announce()`. Reduced motion is honored from `prefers-reduced-motion` and can be toggled, and the `reduced-motion` body class controls it. Keep these working when changing effects.
- Touch and desktop input must both keep working. Any new control needs a touch path as well as a keyboard or mouse path.
- Persisted settings (best score, mute, difficulty, reduced motion) use the `skyfighter.*` keys. Keep the names stable, because changing them resets saved player data.

## Verifying changes

There is no automated test suite. To check a change:
1. Open `index.html` in a browser, or use the pre-installed Playwright Chromium if you are working headless.
2. Start a game, then exercise the path you changed: keyboard, touch controls, pause, bomb, boss fight, and game over.
3. Check the browser console for errors. Test on a narrow viewport as well, because the layout must work at phone width.

State in the summary what you actually ran and what you only checked by reading the code.

## Known gaps

- `localStorage` reads and writes are not wrapped in `try/catch`. In a browser with storage blocked, the game can throw at startup. Wrap them if you touch that code.
- `README.md` has no real project description. Update it if the project's purpose or usage changes.

## Git

- Develop on the branch assigned for the session. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
