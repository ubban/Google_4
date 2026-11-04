# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single HTML5 canvas page.

Files:

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (one IIFE in an inline `<script>`). About 1,700 lines.
- `README.md` — a placeholder title only.
- No build system, package manager, test suite, or linter is configured.

## Running it

No build step. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```sh
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Architecture of `index.html`

The script is wrapped in a single `(function () { 'use strict'; ... })();` closure. Nothing is exported to `window`, so keep new code inside that closure.

Major sections, in file order (search for the function name to jump to it):

- **DOM references** — cached `getElementById` lookups at the top of the script.
- **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`.
- **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to keep frame rate up.
- **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `THEMES` (per-stage sky palette), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage), `KILLS_PER_STAGE`, and `BOSS_PATTERNS`. Most balance changes belong here rather than in the logic.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and `EXPLODE_SFX`, built on the Web Audio API. Audio is created lazily on the first user gesture; `ensureAudio()` must run from a click or key handler.
- **Input** — keyboard (`keydown`/`keyup` on `window`), touch joystick and fire/bomb buttons, and mouse fire. Keyboard bindings: move `WASD` / arrows, hold `Space` to fire, `X` / `Shift` = bomb, `P` / `Esc` = pause. Touch uses a joystick plus fire and bomb buttons; mouse fires by holding the fire button.
- **Game state and logic** — `resetGame`, `spawnEnemy`, `spawnBoss`, `update(dt)`, `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`.
- **Boss attacks** — the `fire*` pattern helpers and `updateBoss` / `killBoss`.
- **Rendering** — `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`. Drawing is plain 2D canvas with no offscreen buffers.
- **Main loop** — `loop(now)` clamps `dt` to 50 ms and calls `update` then `draw` while the game is running and not paused. Started with `requestAnimationFrame`.

The logical play field is `BASE_W × BASE_H` = 480 × 720. `resize()` scales the canvas to fit the viewport; write game coordinates in base units, not CSS pixels.

## Conventions

- **Plain JavaScript, no dependencies.** Do not add a framework, bundler, or npm package unless the user asks. Keep everything in `index.html` unless there is a clear reason to split it.
- **Keep the closure.** New top-level names should be `const`/`let`/`function` inside the IIFE, not globals.
- **Use `const` by default**, `let` only when reassigned. Match the existing style: 2-space indent, single quotes, semicolons, arrow functions for callbacks.
- **Tune in tables.** Prefer adjusting `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES` over adding special cases.
- **Respect reduced motion.** Screen shake and similar effects should scale down when the `reducedMotion` flag is set (see `screenShake`), and CSS uses the `reduced-motion` body class to suppress animations.
- **Accessibility hooks exist.** Announcements go through `announce()` into the `srLive` live region. Keep the buttons' `aria-*` attributes and the `hidden` overlay logic consistent when changing UI.
- **Storage keys.** `localStorage` is read and written directly today, with no try/catch. If you add keys, use the existing `skyfighter.*` naming, and consider wrapping access in try/catch since some browsers block storage.

## Working in this repo

- Develop and push on the branch designated for the session; do not push to other branches without permission.
- There are no automated tests. Verify changes by loading `index.html` in a browser (desktop and a narrow/mobile viewport) and playing through: start, fire, bomb, pause, a stage transition, a boss fight, and game over. Check the console for errors.
- When a change alters gameplay values, mention the old and new numbers in the commit message so balance changes are easy to review.
- Keep commits focused. Do not reformat unrelated parts of `index.html`; the file is large and a noisy diff hides real changes.

## Known gaps

- No tests, linting, or CI.
- `README.md` is a placeholder and does not describe the game. Update it if the project grows beyond the single page.
