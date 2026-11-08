# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free static web project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on a single HTML5 `<canvas>`.

Files:

- `index.html` (~1,670 lines): the whole game. CSS is in a `<style>` block, markup in the body, and all JavaScript in one inline `<script>` wrapped in an IIFE with `'use strict'`.
- `README.md`: a one-line title. Keep it accurate if you expand it.
- No build step, package manager, test suite, linter, or CI config exists yet.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No install step is needed.
- There are no automated tests. Verify changes by playing the game in a browser: start, move, fire, bomb, pause/resume, lose all lives, and check the game-over screen. Also check touch controls at phone width when touching input code.
- Don't add a toolchain (bundler, framework, package.json) unless the user asks for one. The project is intentionally a single file.

## Architecture of `index.html`

Everything below lives in the single inline script.

- **DOM references** are grabbed once at the top (`canvas`, `overlay`, HUD elements such as `scoreEl`, `heartsEl`, `bombsEl`, and buttons such as `startBtn`, `pauseBtn`, `muteBtn`, `motionBtn`).
- **Persistence** uses `localStorage` with keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should keep working if storage is unavailable or returns unexpected values.
- **Tuning tables** are plain constants and the main place to balance the game:
  - `DIFFICULTIES`: spawn rate, enemy speed, enemy shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE`: kills needed to advance a stage.
  - `THEMES`: background palettes cycled per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). `minStage` gates when each type appears.
  - `BOSSES`: six boss designs, selected by stage, with `hpMul` scaling.
  - `BOSS_PATTERNS`: bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Power-ups** are spawned by `maybeSpawnPowerUp` with types `life`, `bomb`, and `weapon`. `applyPowerUp` handles them. The weapon cycles through levels 1–3, then switches between `vulcan` and `laser`.
- **Main loop**: `loop(now)` calls `update(dt)` then `draw()` only while running and not paused. `dt` is capped at 0.05 s so tab switches don't cause big jumps. `trackPerf` adapts a `quality` level (0 full, 1 medium, 2 low) from frame times; respect it when adding expensive draw work.
- **Audio** is synthesized with the Web Audio API (`beep`, `noiseBurst`, and the `sfx` object). Audio starts only via `ensureAudio()` from a user gesture. Don't add audio files without a reason.
- **Input** has three paths: keyboard (`keys` map, `window` keydown/keyup), mouse on the canvas, and touch (joystick in `moveZone`, plus `fireBtn` and `bombBtn`). Keep all three working when changing controls.
- **Accessibility**: `announce()` writes to the `srLive` live region. Reduced motion (`reducedMotion`, `body.reduced-motion`) disables screen shake, flashes, and pulses. Don't add motion effects that skip this check.
- **Canvas sizing** uses a fixed logical size `BASE_W × BASE_H` (480 × 720) scaled by `resize()`. Game coordinates should stay in logical units.

## Conventions

- Plain HTML, CSS, and vanilla JS (ES2017+ features already in use are fine: arrow functions, `const`/`let`, template literals, `Array` methods). Don't introduce external CDN libraries or fonts without asking.
- Keep the code in the existing style: 2-space indentation, single quotes in JS, short section comments like `// ---------- Name ----------`, and `const` for tuning data.
- Keep changes minimal and local. Don't reformat the whole file or split it into modules unless asked.
- Put gameplay balance numbers in the existing tables rather than hardcoding them in update logic.
- Don't break the existing `localStorage` keys; a rename would silently reset players' best score and settings.

## Git workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-3do119`). Don't push to `main` without explicit permission.
- Commit with clear messages that describe the gameplay or UI change.
- Don't open a pull request unless the user asks for one.
