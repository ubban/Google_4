# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a
Raiden-style vertical shoot-em-up, contained in a single self-contained file:

- `index.html` (~1,670 lines): inline `<style>`, markup, and one inline
  `<script>` wrapped in an IIFE (`'use strict'`). Rendering is a 2D canvas.
- `README.md`: a placeholder with no setup instructions.

There is no `package.json`, bundler, framework, test suite, linter, or CI
config. Do not add build tooling unless the user asks for it.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any
  static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a
  browser, and check both desktop (keyboard and mouse) and touch layouts when
  the change touches input or the HUD.

## Code layout inside `index.html`

The script is organized into commented sections. Keep new code in the matching
section:

- **Config constants**: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`,
  `BOSS_PATTERNS`, `KILLS_PER_STAGE`, `BASE_W`/`BASE_H`.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.`
  (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Read and write them
  with the existing constants.
- **Adaptive performance**: `trackPerf` lowers `quality` (0 full, 1 medium,
  2 low) when frame times slip below about 40 fps.
- **Audio**: a Web Audio API synth. `ensureAudio()` creates the context lazily
  on the first user gesture; `beep`, `noiseBurst`, and the `sfx` object make
  the sounds.
- **Input**: keyboard handler on `window`, a virtual joystick for touch, and
  mouse steering on the canvas. The `keys` map is keyed by `KeyboardEvent.code`.
- **Game state**: top-level `let` variables (`score`, `lives`, `stage`,
  `chain`, `multiplier`, and so on). `resetGame()` initializes them.
- **Update and draw**: `update(dt)` runs the simulation and `draw()` renders
  each frame. `loop(now)` drives both through `requestAnimationFrame` and caps
  `dt` at 0.05 s.

## Conventions

- Keep everything in the single `index.html` file unless the user asks for a
  split. Do not introduce modules or external dependencies casually.
- Match the existing style: 2-space indent, single quotes in JavaScript, `const`
  by default, arrow functions for callbacks, and short section banner comments
  of the form `// ---------- Name ----------`.
- Keep the game loop allocation-light. Reuse arrays and objects where the code
  already does, and avoid per-frame closures in hot paths.
- Respect `reducedMotion`. Screen shake, flashes, and pulses must go through the
  existing helpers (`screenShake`, `flashHit`, and so on) so the setting applies.
- Accessibility: announce state changes through `announce()` (the `#srLive`
  live region). Buttons need `title` or visible text labels.
- Balance numbers live in the config tables. Prefer tuning those over
  hard-coding new magic numbers in update logic.

## Gotchas

- Pausing works through `togglePause()`, bound to `P` and `Esc`. Respect the
  `paused` and `gameOver` guards when adding any new timer or input.
- Bombs are bound to `X`, `ShiftLeft`, and `ShiftRight`, and to the touch BOMB
  button. Keep all four paths calling `useBomb()`.
- `Space` default is prevented so the page does not scroll.
- `ensureAudio()` must run from a user gesture. Browsers block `AudioContext`
  until then.

## Git

- Develop on the branch the session names. Do not push to `main` directly.
- Write descriptive commit messages in the imperative mood, such as
  `Add boss pattern for WRAITH stage`.
