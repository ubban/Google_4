# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a minimal repository containing **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up built with HTML5 Canvas and vanilla JavaScript.

- `index.html` — the entire game: markup, CSS (in a `<style>` block), and game logic (in one IIFE inside a `<script>` block). Roughly 1,650 lines.
- `README.md` — a one-line title. No build step, package manager, or dependencies.

There is no `package.json`, bundler, test suite, linter, or CI configuration in the repo at present.

## Running the game

No build is required. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture notes

Everything lives inside a single `(function () { 'use strict'; ... })();` closure, so no game state is exposed on `window`. Key sections, in file order:

- **DOM references and storage keys** — elements are looked up once at the top of the script. Persistent settings use `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
- **Config tables** — `DIFFICULTIES` (easy / normal / hard: spawn rates, enemy speed, enemy fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six stage sky palettes, cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss definitions, with `BOSS_PATTERNS` defined later).
- **Audio** — `ensureAudio()` lazily creates the `AudioContext`. Sound effects are synthesized with `beep()` and `noiseBurst()`; the `sfx` object maps named effects to them. Audio is unlocked on the first user gesture.
- **Game state and loop** — `resetGame()` initializes state; `update(dt)` advances the simulation; `draw()` renders; `loop(now)` drives both via `requestAnimationFrame`.
- **Weapons and power-ups** — `fireWeapon()` implements the Vulcan/Laser system with three power levels; `applyPowerUp()` handles weapon level, bomb, and extra-life pickups; `useBomb()` clears the screen and damages bosses.
- **Bosses** — `spawnBoss()`, `updateBoss()`, `killBoss()`, and the `fire*` pattern helpers (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Flow control** — `togglePause()`, `loseLife()`, and `endGame()`. Game over updates the best score when beaten.
- **Input** — keyboard (arrows or WASD to move, Space to fire, X / Shift to bomb, P / Escape to pause), mouse on the canvas, and touch controls (joystick, fire, and bomb buttons).
- **Accessibility** — a reduced-motion toggle adds the `reduced-motion` body class, and `srLive` provides screen-reader announcements via `announce()`.
- **Responsive sizing** — the canvas uses a fixed logical size (`BASE_W = 480`, `BASE_H = 720`) and `resize()` scales it to the viewport.

## Conventions

- Keep the game self-contained in `index.html` unless the user asks for a split. Do not add a build step or external dependencies without a clear reason.
- Preserve the existing style: 2-space indentation, single quotes in JavaScript, `const` by default, and section-level comments where they already exist.
- Game-balance values live in the config tables near the top of the script. Prefer editing those tables over hard-coding numbers in logic.
- Any new persistent setting should use the `skyfighter.` storage-key prefix and be wrapped in a sensible default when `localStorage` is empty.
- Keep the layout working on phone-sized screens. Touch controls and safe-area insets are part of the design.
- Do not remove the reduced-motion path when adding visual effects; new screen shake, flashes, or animations should respect `reducedMotion`.

## Verifying changes

There is no automated test suite. After editing, verify by:

1. Opening `index.html` in a browser and checking the browser console for errors.
2. Playing through start, pause/resume, bomb use, a stage transition, a boss fight, and game over.
3. Checking that the best score, mute, difficulty, and reduced-motion settings persist across reloads.
4. Checking the layout at a narrow (phone) viewport and with touch controls.

A quick syntax check of the inline script can be done without a browser, for example by extracting the `<script>` block to a temporary file and running `node --check` on it.

## Git workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-lio6yc`). Do not push to other branches without explicit permission.
- Use clear, descriptive commit messages that explain the gameplay or UI change.
- Do not open a pull request unless explicitly asked.
