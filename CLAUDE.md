# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

| File         | Purpose                                                                 |
|--------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`).  |
| `README.md`  | Minimal project readme.                                                 |

There is no build step, no package manager, no bundler, no test suite, and no
linter configured. Everything runs by opening `index.html` in a browser.

## Running and checking changes

- **Run:** open `index.html` directly in a browser, or serve the directory with any
  static server (e.g. `python3 -m http.server` from the repo root, then visit
  `http://localhost:8000`).
- **Verify:** there are no automated tests. After a change, load the page and play
  through at least: start, shoot, a bomb, pause/resume, a stage transition, a boss
  fight, game over, and the difficulty and mute buttons. Check the browser console
  for errors.
- **Mobile:** touch controls (joystick and fire/bomb buttons) are shown on touch
  devices. If you touch input or layout code, check a narrow viewport as well as a
  desktop one.

## Architecture of `index.html`

The game lives in one IIFE inside the `<script>` tag. Keep changes inside that
structure rather than introducing globals.

- **DOM references** are grabbed once at the top (`canvas`, `overlay`, HUD elements,
  touch controls, etc.). Look them up by `id`; if you add a control, add its element
  and its lookup in the same place.
- **Persistence** uses `localStorage` with keys prefixed `skyfighter.`:
  `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes are direct,
  so keep any new keys under the same prefix.
- **Config tables** (edit these to tune the game, not the logic):
  - `DIFFICULTIES`: spawn rates, enemy speed and shot chance, starting `lives`,
    starting `bombs`, and `bossHpMul`.
  - `KILLS_PER_STAGE`: kills needed to advance a stage.
  - `THEMES`: per-stage sky gradient, cloud and sun colors, and display name.
  - `ENEMY_TYPES`: `grunt`, `interceptor`, `tank`, with size, hp, speed, colors,
    score value, and the `minStage` they first appear in.
  - `BOSSES`: name, colors, and `hpMul`, one entry per boss.
  - `BOSS_PATTERNS`: bullet pattern functions used by bosses.
- **Game loop:** `loop(now)` computes a clamped `dt` (max 0.05 s), calls `trackPerf`,
  then `update(dt)` and `draw()` while the game is running and not paused. Game
  state changes belong in `update`; rendering belongs in `draw`.
- **Adaptive quality:** `trackPerf` moves a `quality` level (0 full, 1 medium,
  2 low) based on a rolling average of frame time. Honour `quality` in any new
  expensive effect.
- **Audio:** `ensureAudio()` must run from a user gesture before sound plays. Sounds
  come from `beep()`, `noiseBurst()`, and the `sfx` table. Respect `muted`.
- **Reduced motion:** `reducedMotion` (defaulting to the OS
  `prefers-reduced-motion` setting) suppresses screen shake and flashes. Any new
  shake, flash, or strobe effect must check it.
- **Accessibility:** `announce(text)` writes to the `srLive` live region. Use it for
  meaningful state changes such as stage starts, difficulty changes, and pause.
- **Scaling:** the logical canvas is fixed at `BASE_W × BASE_H` (480 × 720).
  `resize()` scales the displayed size to fit the window, capped at 1.6×. Draw in
  logical coordinates only.

## Conventions

- Plain HTML, CSS, and ES2015+ JavaScript. No frameworks, no external dependencies.
  Don't add a library or CDN script without a clear reason.
- Keep the code in the existing IIFE and `'use strict'` style. Use `const` by
  default and `let` only when reassigned.
- Match the existing naming: `camelCase` for functions and variables, `UPPER_SNAKE`
  for constant tables and storage keys.
- Keep the existing comment banners (`// ---------- Section ----------`) and place
  new code in the matching section.
- Don't reformat unrelated code. Keep diffs focused on the task.

## Git workflow

- Develop on the branch the task names. Do not push to other branches without
  explicit permission.
- Write clear, descriptive commit messages that explain *why* as well as *what*.
- Don't open a pull request unless the user asks for one.
