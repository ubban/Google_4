# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file, `index.html` (~1,670 lines, HTML + inline CSS + inline JavaScript). `README.md` is a placeholder with only the title.

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. Nothing is fetched over the network.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Repository layout

- `index.html` - the entire game: markup, `<style>` block, and `<script>` block.
- `README.md` - title only.
- `CLAUDE.md` - this file.

## Code structure of `index.html`

The inline script is a single `(function () { 'use strict'; ... })();` block of plain browser JavaScript, so nothing is exposed on `window`. Key sections, roughly in file order:

- **DOM references** (~line 426): cached `getElementById` lookups for the canvas, HUD, overlays, touch controls, and buttons.
- **Persistence keys** (~line 463): `localStorage` keys prefixed `skyfighter.` for best score, mute, difficulty, and reduced motion. All reads and writes of `localStorage` should go through these constants.
- **Performance scaling** (`trackPerf`, `quality`): frame times are averaged and the `quality` level (0 full, 1 medium, 2 low) is lowered when frames run slow.
- **Game data tables**: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES`, `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES`. Balance changes usually belong here.
- **Audio**: a lazily created Web Audio context (`ensureAudio`) with helpers `beep`, `noiseBurst`, and a named `sfx` table. Audio is created only after a user gesture.
- **HUD rendering**: `renderHearts`, `renderBombs`, combo pop-ups, stage banners, screen shake, and hit/bomb flashes.
- **Canvas sizing**: fixed logical size `BASE_W` x `BASE_H` = 480 x 720, scaled by `resize()` to fit the window (capped at 1.6x).
- **Input**: keyboard (`keys` map), touch joystick plus fire and bomb buttons, and mouse (which aims at the pointer on the canvas).
- **Game loop, entities, collisions, bosses, pause, and game-over flow** are in the later part of the script.
- **Start button** (`startBtn` click handler, ~line 1655) begins a run.

Accessibility features to keep working: an `aria-live` region (`srLive`, via `announce()`), a reduced-motion toggle that also respects `prefers-reduced-motion`, and a mute toggle.

## Conventions

- Keep the game as a single file unless the user asks for a restructure. Do not add a build system, framework, or package manager without asking.
- Use plain ES2015+ JavaScript with no transpilation. Match the existing style: 2-space indentation, single quotes, `const`/`let`, and short helper functions.
- Keep the layout working at phone width and with touch input. The viewport meta disables zoom, and `touch-action: none` is set on the canvas and body.
- Do not add external scripts, fonts, or network calls. The page is designed to work offline from the file alone.
- Any new persistent value should get a `skyfighter.`-prefixed key next to the existing ones, and reads should tolerate missing or malformed values.

## Verifying changes

There are no automated tests or linters configured. To check a change:

1. Open `index.html` in a browser (or serve it locally) and play through a few stages.
2. Check desktop controls (keyboard and mouse) and, if possible, touch controls at a phone-sized viewport.
3. Confirm that the best score, mute, difficulty, and reduced-motion settings persist across reloads.
4. For a quick syntax check of the inline script, you can extract it and run `node --check` on the extracted file.

## Git

- Develop on the branch assigned for the session. Do not push to other branches without explicit permission.
- Do not create pull requests unless the user asks for one.
