# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free static web project. Its only content is
**Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page
HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`, roughly lines 10–361),
  HUD/overlay/touch-control markup (roughly lines 362–421), and one IIFE in a
  `<script>` block (roughly lines 422–1669) that holds all game logic.
- `README.md` — placeholder; no setup instructions.

There is no build step, package manager, test suite, linter, or CI configuration.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server`). No install is required.

## Architecture of the game script

The script is wrapped in a single `(function () { 'use strict'; ... })()`, so
nothing leaks to the global scope. Keep new code inside that wrapper.

Key sections, in file order:

- **DOM references** — all `document.getElementById` lookups are grabbed at the
  top of the IIFE.
- **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`,
  `STORAGE_REDUCED`, all under the `skyfighter.*` prefix in `localStorage`.
  Reads and writes are not wrapped in `try/catch`; if you add more storage use,
  guard it, since storage can be blocked in some browsers.
- **Adaptive quality** — `trackPerf` samples frame times and steps `quality`
  (0 full, 1 medium, 2 low) to keep frame rate up on slow devices.
- **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`,
  `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here rather than in
  the update logic.
- **Audio** — Web Audio synthesis only (`beep`, `noiseBurst`, named `EXPLODE_SFX`
  mappings). No audio files are loaded. `ensureAudio()` must run from a user
  gesture before sound plays.
- **Input** — keyboard (`keys[e.code]`), mouse on the canvas, and touch via the
  joystick (`moveZone`) and the FIRE/BOMB buttons. Keep all three working when
  you change controls.
- **Game state and loop** — `resetGame()` initializes state; `update(dt)`
  advances it; `draw()` renders it; `loop(now)` drives both through
  `requestAnimationFrame`, with `dt` clamped to 50 ms. Pause is handled by
  `running`/`paused`.
- **Boss system** — `BOSS_PATTERNS` are bullet-pattern functions (`fireRadial`,
  `fireAimedSpread`, `fireSpiralBurst`, etc.) selected by `updateBoss`.
- **Rendering helpers** — `drawPlane`, `drawBossShip`, plus the canvas-based
  particle, explosion, and power-up drawing.

Canvas is a fixed logical size of `BASE_W × BASE_H` (480 × 720), scaled to fit
the viewport in `resize()`. Write game coordinates against that base size.

## Conventions

- Match the existing style: two-space indentation, single quotes in JS,
  `const`/`let` (no `var`), arrow functions for callbacks, and `camelCase`
  identifiers. Constants use `UPPER_SNAKE_CASE`.
- Keep the file self-contained. Do not add external scripts, stylesheets, or
  fonts unless the user asks for them.
- Keep comments sparse and in the style of the existing `// ---------- Section ----------`
  headers.
- Accessibility matters here: respect the reduced-motion setting
  (`body.reduced-motion`, `reducedMotion`) when adding screen shake, flashes,
  or pulses, and announce important state changes through `announce()`.
- Keep gameplay tuning in the constant tables rather than hard-coding numbers in
  update logic.

## Verifying changes

There are no automated tests. To verify a change:

1. Open `index.html` in a browser (or serve it locally).
2. Check the affected flow end to end: start, play, pause/resume, bomb, boss
   fight, game over, and restart.
3. Check all three difficulty levels if you changed balance values.
4. Check touch controls with a touch-emulating browser mode if you changed input.
5. Confirm there are no console errors.

If you cannot run the game in the environment, say so explicitly and verify by
reading the code.

## Git

- Develop on the branch specified by the session, and push with
  `git push -u origin <branch>`.
- Write commit messages in the imperative mood with a short summary line, plus
  a body when the reasoning is not obvious from the diff.
- Do not open pull requests unless the user asks for one.
