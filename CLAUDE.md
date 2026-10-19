# CLAUDE.md

Guidance for working in this repository.

## What this is

`Google_4` is a small static web game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up played on an HTML canvas. The whole game lives in one file, `index.html` (about 1,670 lines: inline `<style>`, markup, and one inline `<script>`). `README.md` is a placeholder.

## Repository layout

- `index.html` — the entire game: CSS, DOM overlay/HUD, canvas, and game logic.
- `README.md` — minimal project title.
- `CLAUDE.md` — this file.

There is no `package.json`, bundler, test suite, linter, or CI config. Do not add tooling unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install step is needed.

## Code structure inside `index.html`

The script is one IIFE with `'use strict'`. Keep new code inside it so globals stay clean.

- **DOM refs** — all `getElementById` / `querySelector` lookups are grouped at the top of the script.
- **Storage keys** — `STORAGE_*` constants use the `skyfighter.*` prefix (best score, mute, difficulty, reduced motion).
- **Tuning tables** — `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, and `KILLS_PER_STAGE` hold balance values. Change numbers there rather than hard-coding them in logic.
- **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object are built on the Web Audio API. Audio starts only after a user gesture (`ensureAudio()` is called from input handlers).
- **Input** — keyboard (`keys[e.code]`), touch joystick plus fire/bomb buttons, and mouse (`canvas` move and mousedown). New controls should go through the same `keys` / `firing` / `mouseActive` state.
- **Game state** — `resetGame()` initializes state; `update(dt)` advances the simulation; `draw()` renders. `loop(now)` runs `trackPerf`, then `update`, then `draw`, while the game is running and not paused.
- **Adaptive quality** — `trackPerf()` steps a `quality` level (0–2) from frame timings. Drawing code should respect it so low-end devices stay smooth.
- **Persistence** — best score is written only when a run ends (`endGame()`). Everything else is saved on change.
- **Accessibility** — `announce()` writes to the `srLive` live region. `reducedMotion` (user setting, defaulting to `prefers-reduced-motion`) toggles the `reduced-motion` body class. `screenShake()` scales magnitude down and caps duration when it is on, so new flash or shake effects should go through the same path.
- **Responsive scaling** — the logical play field is `BASE_W` × `BASE_H` (480 × 720); `resize()` scales the canvas to fit the viewport.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for handlers, short comment banners (`// ---------- Section ----------`) to separate sections.
- Keep the game framework-free. Vanilla JS, one file, no external dependencies unless the user asks for one.
- Keep frame-rate independence: movement and timers use `dt` (seconds, clamped to 0.05 in the loop).
- Pause must freeze everything that `update()` drives. Check `paused` and `running` when adding timed effects.
- Do not break the mobile layout: the page uses `viewport-fit=cover` and touch controls. Test changes to the HUD or overlay at phone width.

## Verifying changes

Since there are no automated tests, verify by running the game in a browser:

1. Start a run, then check movement, firing, bombs, power-ups, and pause.
2. Check each difficulty, and that the best score persists after a reload.
3. Check mute and reduced-motion toggles, and touch controls at a narrow viewport.
4. Check the browser console for errors during play.

For changes to balance tables, play long enough to reach a boss and at least one stage transition.

## Git

- Develop on the branch you were asked to use (currently `claude/gifted-tesla-7nkkox`).
- Write clear, descriptive commit messages that explain the gameplay or code change.
- Do not open pull requests unless asked.
