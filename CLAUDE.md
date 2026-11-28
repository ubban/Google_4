# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up), with no build step and no dependencies.

- `index.html` — the entire game: markup, CSS (`<style>` block, lines 10–359), canvas element, and one inline `<script>` IIFE (roughly lines 422–1668) containing all game logic.
- `README.md` — minimal placeholder.

There is no package.json, test suite, linter, or bundler. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No install step.

## Architecture notes (inside the `<script>` IIFE)

- **Fixed logical resolution:** `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to the viewport. Game coordinates always use the 480×720 space.
- **Game loop:** a single `requestAnimationFrame` `loop(now)` calls `update(dt)` then `draw()` while `running && !paused`. `dt` is clamped to 0.05s.
- **Adaptive quality:** `trackPerf()` samples frame times over 45 frames and steps a `quality` level (0–2) up or down to shed effects on slow devices.
- **Difficulty:** `DIFFICULTIES` table (`easy`/`normal`/`hard`) controls spawn rates, enemy speed and fire chance, starting lives and bombs, and boss HP multiplier.
- **Content tables:** `THEMES` (stage backgrounds, cycled per stage; a stage is `KILLS_PER_STAGE = 18` kills), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses), `BOSS_PATTERNS` (bullet patterns).
- **Power-ups:** spawned in `maybeSpawnPowerUp()` with types `life`, `bomb`, `weapon`; applied in `applyPowerUp()`.
- **Audio:** Web Audio synthesis only (`beep`, `noiseBurst`, `sfx.*`). No audio files. `ensureAudio()` must run from a user gesture before sound plays.
- **Accessibility:** `announce()` writes to the `#srLive` ARIA live region. Reduced motion is detected from `prefers-reduced-motion` and can be toggled in-game; when on, `screenShake` and flash effects are dampened or skipped.
- **Persistence:** `localStorage` keys under the `skyfighter.*` prefix (`bestScore`, `muted`, `difficulty`, `reducedMotion`). All reads/writes are unguarded today, so keep that in mind if you add more (see "Conventions").
- **Input:** keyboard (arrows/WASD move, Space fires, `P`/`Esc` pause, `X`/`Shift` bomb), mouse on the canvas, and a touch joystick plus fire/bomb buttons shown on touch devices.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a restructure. Don't split files or add a build step as a side effect of another change.
- Stay in the existing style: `const`/`let`, 2-space indentation, single quotes in JS, and everything inside the IIFE (no new globals).
- Add new tunables to the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in logic.
- Preserve the `'use strict'` IIFE wrapper and the `reducedMotion` checks when adding visual effects.
- Anything that touches `localStorage` should be wrapped in try/catch so the game still runs in private mode or with blocked storage.
- Keep the layout usable at phone width; the touch controls are part of the game, not an afterthought.

## Verifying changes

There are no automated tests. For any gameplay or UI change:

1. Open `index.html` in a browser (or use the `run` skill / Playwright) and start a game.
2. Check the affected path on desktop and, if the change touches input or layout, in a touch-sized viewport.
3. Confirm the console has no errors on load and during play.

Say explicitly in the final message when a change was only verified by reading the code.

## Git

- Development branch for this session: `claude/gifted-tesla-ybkr7c`. Push there with `git push -u origin <branch>`.
- Don't open a pull request unless the user asks for one.
- Commit messages: short imperative subject; body optional.
