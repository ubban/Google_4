# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The whole game lives in one file:

- `index.html` (~1,670 lines): inline `<style>`, markup, and one IIFE `<script>` that holds all game logic, rendering, audio, and input.
- `README.md`: one-line title only.

There is no build step, no package.json, no bundler, no test suite, and no linter. Do not add tooling unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verification so far has been manual or headless Playwright runs. The Chromium that Playwright needs is preinstalled at `/opt/pw-browsers/chromium`. Launch it with `--headless --no-sandbox` if you pass an explicit path.
- For any gameplay change, check it in a real run, not just by reading the code: start a game, exercise the changed path, and confirm there are no console errors.

## Code layout inside `index.html`

The script is one IIFE. Sections, roughly in file order:

1. **DOM references**: `const xxxEl = document.getElementById(...)` at the top. Element ids are referenced by name, so renaming an id in the markup means updating the script too.
2. **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all under the `skyfighter.*` prefix. Reads are direct `localStorage` calls, so keep any new key namespaced the same way.
3. **Tuning tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE`: kills needed to advance a stage.
   - `THEMES`: background palettes, cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (multi-hit). Each has `minStage`.
   - `BOSSES`: six designs that cycle by stage.
4. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with a compressor feeding `masterGain`. Helpers `beep`, `noiseBurst`, and `panFor` build layered sounds. Sound effects are named entries in the `sfx` object. Audio must be created from a user gesture, so call `ensureAudio()` in input handlers.
5. **Input**: keyboard (`keys` map, plus P/Esc pause and X/Shift bomb), mouse steering and fire on the canvas, and touch (joystick plus FIRE and BOMB buttons). Keyboard and touch take priority over mouse steering.
6. **Rendering and layout**: `resize()` scales a fixed 480×720 logical canvas (`BASE_W`, `BASE_H`) to fit the window, capped at 1.6×. Drawing uses `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
7. **Game state and simulation**: `resetGame`, `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `useBomb`, `update(dt)`, `updateBoss(dt)`, `endGame`.
8. **Main loop**: `loop(now)` clamps `dt` to 0.05 s, calls `trackPerf`, then `update` and `draw` while the game is running and not paused. The loop is started once with `requestAnimationFrame(loop)`.

Everything is inside the IIFE, so nothing is a global. Keep new code inside it unless there is a reason to expose it.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, `let` only when reassigned, and short comments that explain *why*.
- Keep the file a single self-contained HTML file. No external network dependencies, and no new `<script src>` or `<link>` tags.
- Use the `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` tables for tuning values instead of hard-coding numbers into update or draw logic.
- Respect the accessibility and performance hooks already in place:
  - `reducedMotion` should dampen screen shake and disable flash and pulse effects. New visual effects that flash or shake should check it.
  - The `quality` tier (0 to 2) trims particle counts and shadow-blur glow. New particle or glow effects should respect it.
  - Announcements for screen readers go through `announce(text)` and the `#srLive` region.
- Touch and mouse support both matter. Any new control needs a keyboard binding and, where it makes sense, a touch equivalent.
- Keep `touch-action: none` and the no-scroll layout intact.

## Git

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-y251zu`). Do not push to `main` unless explicitly asked.
- Write commit messages in the style of the existing history: a short imperative summary line, then a body that explains what changed and how it was verified.
- Do not open a pull request unless the user asks for one.
