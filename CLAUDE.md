# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–362), and JavaScript (`<script>`, ~line 422 onward). |
| `README.md` | Placeholder README (project title only). |

There is no build system, no package manager, no bundler, no tests, and no
dependencies. The game runs by opening `index.html` in a browser.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server
  (e.g. `python3 -m http.server`) and visit it.
- There is no automated test suite or linter. Verify changes by loading the page and
  playing: start a game, check firing, bombs, pause, difficulty buttons, the boss fight,
  and the game-over overlay. Check both desktop (keyboard) and touch layouts when
  touching input or HUD code.
- For visual checks, use the pre-installed Playwright/Chromium (no `playwright install`).

## Architecture of `index.html`

- **Single IIFE**: all game code lives inside `(function () { 'use strict'; ... })();`.
  Do not introduce globals; keep new code inside the IIFE.
- **Rendering**: one `<canvas id="gameCanvas">` drawn with the 2D context. Logical
  resolution is `BASE_W = 480`, `BASE_H = 720`; `resize()` scales it to the viewport.
- **Game loop**: `requestAnimationFrame` → `loop(now)` → `update(dt)` then `draw()`.
  `dt` is clamped to 0.05 s. Update logic and drawing stay separate; don't mutate state in `draw*`.
- **Adaptive quality**: `trackPerf()` samples frame times and steps `quality` (0 full,
  1 medium, 2 low) to reduce effects when frames are slow.
- **Tuning data** lives in top-of-script constant tables. Prefer editing these over
  hard-coding numbers in logic:
  - `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier)
  - `KILLS_PER_STAGE` (kills needed to advance a stage)
  - `THEMES` (per-stage sky/cloud/sun palettes)
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`)
  - `BOSSES` (boss names, colors, HP multiplier) and `BOSS_PATTERNS` (attack patterns)
- **Entities** are plain objects in arrays (enemies, bullets, power-ups, explosions),
  updated and filtered each frame. Collisions use `rectsOverlap()`.
- **Audio** is generated with the Web Audio API (`beep`, `noiseBurst`, `ensureAudio`);
  there are no audio files. Audio must start from a user gesture (`ensureAudio()`).
- **Accessibility**: `announce()` writes to the `#srLive` live region. Respect the
  reduced-motion setting (`reducedMotion`, `body.reduced-motion`) for new screen shake,
  flashes, or animations.

## Persistence

Settings and scores use `localStorage` with these keys (defined as `STORAGE_*` constants):

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Renaming or changing the meaning of a key silently resets players' saved data. Keep keys
stable, or add migration logic if a change is unavoidable.

## Controls (for reference when changing input)

- **Keyboard**: arrow keys / WASD move (via `keys[e.code]`), Space fires, `X` / Shift
  uses a bomb, `P` / Escape pauses.
- **Touch**: virtual joystick in `#moveZone`, `#fireBtn` (hold to fire), `#bombBtn`.
  Touch controls are shown only on touch devices (`isTouchDevice`).

## Conventions

- Keep the project dependency-free: plain HTML, CSS, and JavaScript only.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`,
  arrow functions for callbacks, and descriptive section comments (`// ---------- Name ----------`).
- Keep the game in `index.html` unless the user asks for a split. If you do split files,
  update this document.
- Do not add a build step, framework, or package manager without the user's approval.

## Git

- Development branch for this work: `claude/gifted-tesla-7c66rc`. Do not push to other
  branches without explicit permission.
- Do not open pull requests unless the user asks.
- Write clear commit messages in the imperative mood, with a short subject and a
  descriptive body when the change is non-trivial.

## Keeping this file current

Update `CLAUDE.md` when you change the file layout, add or rename storage keys, change
the controls, or introduce new tooling or conventions. Keep it short and factual.
