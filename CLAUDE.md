# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML5 canvas page.

Files:

- `index.html` — the entire game: markup, inline CSS (`<style>`), and inline JavaScript (`<script>`). No build step, no dependencies, no external assets.
- `README.md` — a one-line title. Keep it minimal unless asked to expand it.

There is no package manager, test suite, linter, or CI configuration in the repo.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Nothing needs to be installed. For automated checks, a headless Chromium with Playwright works (see the environment notes for the pre-installed browser).

## Code layout inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Keep new code inside it so no globals leak.

Key sections, roughly in file order:

- **DOM refs** — `getElementById` lookups at the top of the IIFE. If you add a HUD or overlay element, add its ref here.
- **Constants** — `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky gradients per stage), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
- **Persistence** — `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes are not wrapped in try/catch; wrap them if you touch that code, since storage can be unavailable in private windows.
- **Audio** — a lazily created `AudioContext` (`ensureAudio`) and synthesized sound effects (`beep`, `noiseBurst`, `sfx`). No audio files.
- **Layout** — `BASE_W = 480`, `BASE_H = 720`. The canvas is a fixed-size backing store scaled by CSS in `resize()`. Game coordinates always use the base size.
- **Input** — keyboard (`keys` map keyed by `KeyboardEvent.code`), mouse (steer and click-to-fire on the canvas), touch (virtual joystick, FIRE and BOMB buttons), and `firing` for held fire.
- **Game state** — `resetGame()` initializes `player`, enemies, bullets, power-ups, boss, score, and chain multiplier. `update(dt)` advances the simulation. `draw()` renders it. `loop(now)` is the `requestAnimationFrame` driver.
- **Adaptive quality** — `trackPerf()` watches frame time and steps a `quality` level down or up. Drawing code should respect it rather than always using the most expensive effect.
- **Accessibility** — a `reducedMotion` setting, and `srLive` for screen-reader announcements via `announce()`.

Controls (also shown in `#controlsHint`): mouse to steer and click to fire; arrows or WASD to move; Space to fire; X or Shift for a bomb; P or Esc to pause.

## Conventions

- **Stay in one file.** The project is deliberately a single self-contained page. Do not split out modules, add a bundler, or add npm dependencies unless the user asks.
- **No external network requests** from the game. Keep fonts, scripts, and images inline or generated in code.
- **Use `dt`-based movement** in `update()` so the game plays at the same speed regardless of frame rate.
- **Respect `reducedMotion` and `quality`** when adding screen shake, particles, or other heavy effects.
- **Keep touch and keyboard parity.** Any new player action needs a keyboard binding and a touch control, and should be mentioned in `#controlsHint`.
- **Preserve existing style:** two-space indentation, single quotes in JS, short helper functions, and comments only where the intent is not obvious.
- **Mobile first.** The page locks scrolling and uses `touch-action: none`. Test layout at phone width, including safe-area insets.

## Making changes

- Make the smallest change that does the job. Don't reformat unrelated code.
- After editing `index.html`, load it in a browser and check the console for errors. Start a game, play briefly, pause and resume, and trigger a bomb if the change touches gameplay.
- Check both light and dark system themes if you change CSS; the page sets `color-scheme: dark`.
- Commit on the branch the task names. Don't push to other branches without explicit permission.

## Things to avoid

- Adding a framework, build tool, or package manifest without a request.
- Hard-coding pixel sizes that ignore `BASE_W`/`BASE_H` scaling.
- Blocking calls such as `alert()` or synchronous `localStorage` loops inside the game loop.
- Editing `README.md` or other files as a side effect of a gameplay change.
