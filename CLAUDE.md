# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up that runs in the browser on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup, CSS (`<style>`, lines 10–359), and JavaScript (`<script>`, ~lines 422–1667). No external dependencies.
- `README.md` — a one-line title. Keep it minimal.
- There is no build system, package manager, bundler, test suite, or linter.

## Running the game

No build step is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Repository layout and conventions

- The game is a single self-contained file. Keep CSS, markup, and JS in `index.html` unless the user asks for a split. Do not introduce a framework or build tooling without an explicit request.
- JavaScript is wrapped in one IIFE with `'use strict'`. Do not add globals; keep new state inside that closure.
- Code is plain ES2015+ (`const`/`let`, arrow functions, template-free DOM APIs). Match that style.
- Sections in the script are marked with `// ---------- Name ----------` comments. Follow that convention when adding a new section.
- DOM lookups happen once at the top of the script, using `getElementById` / `querySelectorAll`. Add new element references there, not inline.

## Key subsystems in `index.html`

- **Config tables** (near the top of the script): `DIFFICULTIES` (spawn rates, lives, bombs, boss HP multiplier), `THEMES` (sky palettes, one per stage cycle), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss definitions with `hpMul`). Tune gameplay here rather than in logic code.
- **Stages**: `KILLS_PER_STAGE` controls how many kills advance a stage. Bosses spawn via `spawnBoss()`.
- **Persistence** (`localStorage`): keys are prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes are not wrapped in try/catch today; if you touch them, consider guarding against storage being blocked.
- **Audio**: synthesized with the Web Audio API (`beep`, `noiseBurst`, the `sfx` object). Audio is created lazily by `ensureAudio()`, which must run from a user gesture.
- **Adaptive quality**: `trackPerf()` samples frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low). Respect this when adding expensive effects.
- **Game loop**: `loop(now)` clamps `dt` to 50 ms, calls `update(dt)` then `draw()` while running and not paused, and is driven by `requestAnimationFrame`. `update` handles movement, collisions, spawning, and power-ups; `draw` renders everything.
- **Weapons and power-ups**: `fireWeapon()`, `applyPowerUp()`, and `useBomb()`. Weapon levels range over the Vulcan/Laser system; bombs are limited per difficulty.
- **Boss battles**: `BOSS_PATTERNS` is an array of attack functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, etc.). `updateBoss()` cycles through them; `killBoss()` and `loseLife()` handle outcomes.
- **Input**: keyboard (arrows/WASD move, Space fire, X/Shift bomb, P/Esc pause), mouse on the canvas, and touch controls (`joystick*`, `fireBtn`, `bombBtn`) for mobile. Changes to controls must update the `#controlsHint` text.
- **Accessibility**: `announce()` writes to the `#srLive` live region. Reduced motion (user toggle plus `prefers-reduced-motion`) sets the `reduced-motion` body class, which disables the CSS flash, vignette, combo, and banner animations. `screenShake()` checks the flag directly and scales shake down rather than removing it.

## Making changes

- Read the relevant section of `index.html` before editing; the file is large and sections are tightly coupled through shared state variables.
- Keep the canvas logical size (`BASE_W` × `BASE_H`, 480×720) in mind. `resize()` scales it to the viewport, so do not hard-code pixel positions outside that coordinate space.
- Preserve mobile layout. The game must work at phone width with touch controls; verify any HUD or CSS change at a narrow viewport.
- Do not remove the `prefers-reduced-motion` handling or the screen-reader announcements when refactoring.

## Verification

There are no automated tests or linters. Verify changes by:

1. Opening `index.html` in a browser (desktop and a narrow/mobile viewport) and playing through at least one stage and a boss fight.
2. Checking the browser console for errors.
3. Confirming persistence still works: best score, mute, difficulty, and reduced-motion survive a reload.
4. For a syntax-only check of the inline script, extracting the `<script>` block and running `node --check` on it is acceptable.

If you cannot run the game in a browser, say so explicitly in your summary rather than claiming the change was tested.

## Git and branches

- Develop on the branch specified by the task (currently `claude/gifted-tesla-g50hc1`). Do not push to `main` without explicit permission.
- Write commit messages in the imperative mood with a short summary line, followed by a body explaining *why* when it is not obvious.
- Do not open a pull request unless the user asks for one.
