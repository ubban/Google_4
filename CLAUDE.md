# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 `<canvas>`. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), and JavaScript (`<script>`, ~lines 422–1668), wrapped in a single IIFE.
- `README.md` — placeholder; currently contains no project documentation.

To run the game, open `index.html` directly in a browser, or serve the directory with any static server.

## Architecture (inside `index.html`)

The script is one IIFE with these main parts, roughly in file order:

1. **DOM references and storage keys** (`STORAGE_*` constants, `localStorage` for best score, mute, difficulty, reduced motion).
2. **Config tables**
   - `DIFFICULTIES` — `easy` / `normal` / `hard`: spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier.
   - `THEMES` — per-stage sky gradient, cloud/sun colours and name (cycled by stage).
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaves), `tank` (4 HP); `minStage` gates when each appears.
   - `BOSSES` — six boss designs, one per stage cycle, each with its own `hpMul`.
   - `KILLS_PER_STAGE` (18) — kills needed before a boss warning.
3. **Audio** — Web Audio synth (`beep`, `noiseBurst`, the `sfx` map). Audio context is created lazily by `ensureAudio()`, which must run from a user gesture.
4. **Rendering helpers** — `drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion`, screen shake, flash and vignette overlays.
5. **Input** — keyboard (`keys[e.code]`), mouse (steer and fire on canvas), touch joystick plus on-screen FIRE and BOMB buttons. `resize()` scales the fixed 480×720 logical canvas (`BASE_W`/`BASE_H`) to fit the window.
6. **Game state and simulation** — `resetGame()` initialises state; `update(dt)` advances player, bullets, enemies, power-ups, collisions, chain/multiplier and stage progression; `updateBoss(dt)` drives boss patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`); `endGame()` records the best score and shows the game-over overlay.
7. **Drawing** — `draw()` renders each frame.
8. **Main loop** — `loop(now)` is a `requestAnimationFrame` loop. `dt` is clamped to 50 ms. `trackPerf()` adjusts a `quality` level (0 full, 1 medium, 2 low) when frame times are poor.

### Gameplay facts worth knowing

- Logical playfield is 480×720. Keep gameplay coordinates in that space, not in CSS pixels.
- Power-ups (`life`, `bomb`, `weapon`) drop from enemies. The `weapon` power-up toggles between `vulcan` and `laser`.
- Player starts with `lives` and `bombs` from the chosen difficulty. Bombs are used with X or Shift, or the on-screen BOMB button.
- Pause is P or Escape.
- Reduced motion can be set manually and is also read from `prefers-reduced-motion`.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Don't introduce a bundler, framework, or npm dependencies without asking.
- **No external requests.** The page currently loads nothing remote; the favicon is an inline SVG data URI. Keep it that way.
- **Match the surrounding style.** The code uses 2-space indentation, single quotes in JS, `const`/`let` (no `var`), small named functions inside the IIFE, and config as plain object literals at the top of the script. Follow these when adding code.
- **Game-balance numbers live in the config tables.** Prefer tuning `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES` over hard-coding values inside `update()` or `draw()`.
- **Persist only through the existing `STORAGE_*` keys**, and read/write them the same way the current code does.
- **Accessibility hooks** — keep the `srLive` announcer (`announce()`), `aria-*` attributes, and reduced-motion checks working when changing UI or effects.
- **Mobile must keep working.** Touch controls are shown on touch devices; test layout changes at phone width.

## Working on this repo

- There is no build, lint, or test command. Verify changes by opening `index.html` in a browser (or a headless browser) and playing through the affected path: start, fire, bomb, pause, stage transition, boss, game over, and difficulty change.
- Check the browser console for errors after any JS change.
- Keep changes minimal and scoped to the request; this is a small project, so avoid speculative refactors.
- When you change game-visible behaviour (controls, scoring, balance, stage flow), update the on-screen `#controlsHint` text and this file if the architecture description changes.

## Git

- Default branch work happens on the branch named in the session instructions; don't push to other branches without permission.
- Write descriptive commit messages focused on the why.
