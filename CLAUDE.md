# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one self-contained file, `index.html` (~1,670 lines: inline `<style>` plus one inline `<script>`, drawn on a `<canvas>`). There is no build step, no package manager, no bundler, and no external assets. Audio is synthesized with the Web Audio API.

Files:

- `index.html` — the entire game (markup, CSS, and JavaScript).
- `README.md` — a one-line title only.
- `CLAUDE.md` — this file.

Git: the initial commit is `4efdad1`; the game was added in `c4dc8f6`.

## Running and testing

- Run it by opening `index.html` in a browser, or serve the directory with any static server (for example `python3 -m http.server` and visit `http://localhost:8000`).
- There is no test suite, linter, or build. Verify changes by playing the game in a browser. For automated checks, use the pre-installed Chromium/Playwright described in the environment notes (do not reinstall Playwright).
- Test both desktop (keyboard and mouse) and touch layouts when changing input, HUD, or layout code. The page is designed to scale to any viewport (see `resize()`).

## Code layout (inside `index.html`)

The script is a single `(function () { 'use strict'; ... })()` block starting at about line 423, and it reads DOM elements by id at the top. Key sections, in file order:

| Area | Where to look |
| --- | --- |
| DOM references | Top of script: `const canvas`, `overlay`, `hudEls`, etc. |
| Persistence keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `localStorage`, prefixed `skyfighter.`) |
| Performance scaling | `quality` / `trackPerf()` — drops visual quality when frame times are slow |
| Difficulty table | `DIFFICULTIES` — `easy`, `normal`, `hard`; sets spawn rate, enemy speed, shoot chance, starting lives/bombs, boss HP multiplier |
| Stage data | `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes that rotate by stage), `ENEMY_TYPES`, `BOSSES` (six bosses, selected by `(stage - 1) % 6`; `THEMES` rotates the same way) |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx` object. Audio context is created lazily on the first user gesture. |
| HUD and feedback | `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake` |
| Layout | `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit (max scale 1.6) |
| Input | `keys` map, keyboard listeners, joystick (`joystickStart/Move/End`), fire/bomb touch buttons, mouse steering and click-to-fire |

Controls (also shown in the in-game hint, `#controlsHint`):

- Keyboard: Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mouse: move to steer, click to fire.
- Touch: drag left zone to fly (virtual joystick), tap FIRE, tap BOMB.

## Conventions

- **Keep it a single file.** New game code, styles, and markup go into `index.html` unless the user asks for a split. Do not add a build system, framework, or npm dependencies without an explicit request.
- **No external network dependencies.** The game is offline-capable; keep the favicon and assets inline (data URIs / canvas drawing).
- **Preserve the visual style.** Colors are hard-coded hex/rgba values in the CSS and in the `THEMES`, `ENEMY_TYPES`, and `BOSSES` tables. Reuse the existing palette and naming style (camelCase for JS, kebab/camel ids for DOM).
- **Respect accessibility settings.** The game honors `prefers-reduced-motion` (`reducedMotion`, `body.reduced-motion`) and has an `aria-live` region (`srLive`) that `announce()` writes to. New animations should go through the reduced-motion check; new important game events should be announced.
- **Respect the mute setting.** Every sound effect should go through `sfx` / `beep` / `noiseBurst`, which already respect `muted`.
- **Keep the game loop allocation-light.** Prefer reusing arrays and objects in per-frame code; the `quality` mechanism exists because low-end devices drop frames.
- **Mobile first on layout.** Keep the touch controls and the HUD usable at small widths; `touch-action: none` and `user-select: none` are intentional on the body.

## Making changes

- Read the surrounding section before editing; the script is one large block and state is shared through top-level `let`/`const` variables.
- When adding a stage, enemy, boss, or theme, extend the existing data table rather than hard-coding special cases in the loop.
- When changing difficulty, update the `DIFFICULTIES` table so all three levels stay consistent.
- Keep `localStorage` access tolerant of failure (private windows and blocked storage can throw); wrap new reads and writes in try/catch.
- After editing, open the game in a browser and check: start, a full stage (the boss warns after 18 stage kills), death/game-over, bomb use, pause (P/Esc), and mute.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-g9ibi0`). Do not push to other branches without permission.
- Use clear, descriptive commit messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.
