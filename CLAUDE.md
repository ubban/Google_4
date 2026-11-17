# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up written as a single self-contained HTML5 canvas page.

- `index.html` (~1670 lines): the entire game — markup, CSS (`<style>`), and JavaScript (`<script>`). No build step, no dependencies, no framework.
- `README.md`: placeholder. Keep it accurate if you change how the project is run.

There is no package manager, test suite, linter, or CI config in the repo today.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server
  (e.g. `python3 -m http.server` from the repo root) and visit `http://localhost:8000`.
- There are no automated tests. Verify gameplay changes by running the page in a browser
  and exercising the affected path (title screen, a stage, a boss, game over, pause, mute,
  difficulty switch). Check the browser console for errors.
- Test both desktop (keyboard/mouse) and touch layouts when touching input or HUD code.
  Touch controls appear only when `'ontouchstart' in window` or `navigator.maxTouchPoints > 0`.

## Code layout inside `index.html`

The script is one inline block wrapped in an IIFE (`(function () {` at ~line 423), so nothing
is exposed globally. Sections, grouped roughly in this order:

1. **Constants** (~lines 460–520): `STORAGE_*` localStorage keys, `DIFFICULTIES`
   (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier),
   `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`.
2. **Persisted settings**: best score, mute, difficulty, reduced motion. Stored under
   `skyfighter.*` keys.
3. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, `panFor`. Audio is created lazily on the
   first user gesture (browsers block autoplay), so keep calls to `ensureAudio()` on input handlers.
4. **UI helpers**: hearts/bombs rendering, flashes, combo pops, stage banner, screen shake,
   `announce()` for the `aria-live` region.
5. **Input**: `keydown`/`keyup` on `window`, virtual joystick and buttons for touch,
   mouse steering/firing on the canvas.
6. **Game state and loop**: `resetGame()`, `update(dt)` (main per-frame simulation),
   spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), combat (`fireWeapon`,
   `useBomb`, `applyPowerUp`, `rectsOverlap`), scoring (`addScore`, kill-chain via `breakChain`).
7. **Rendering**: `drawPlane`, `drawBossShip`, and the draw pass for clouds, sun, bullets, and HUD canvas effects.
8. **Boss patterns**: `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns,
   spiral, cross, homing, double ring).

Key controls (also shown in `#controlsHint`): mouse steer/click fire, arrows/WASD move,
Space fire, X or Shift bomb, P or Esc pause.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a
  bundler, framework, or npm dependency without a clear reason and the user's approval.
- **Plain ES2015+ JavaScript and CSS.** Match the existing style: `const`/`let`, arrow functions,
  2-space indentation, single quotes in JS, and `UPPER_SNAKE` for tuned constants.
- **Tuning lives in constants.** Put balance numbers in `DIFFICULTIES`, `BOSSES`, `ENEMY_TYPES`,
  or `THEMES` rather than inlining magic numbers in `update()`.
- **Frame-rate independence.** `update(dt)` takes a delta time. Scale movement and timers by `dt`.
- **Persistence.** All saved state goes through the `STORAGE_*` keys. Wrap new localStorage access
  in a way that tolerates private-mode or disabled-storage errors.
- **Accessibility.** Respect the reduced-motion toggle for shake, flashes, and pulses. Announce
  important state changes through `announce()`.
- **Mobile.** Don't break the touch layout. The canvas scales via `resize()` against `BASE_W` × `BASE_H`
  (480 × 720).

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-yk57eo`).
  Do not push to other branches without explicit permission.
- Do not open a pull request unless the user explicitly asks for one.
- Keep commits focused. Describe the gameplay or UI effect in the message body, not just the file touched.

## Things to avoid

- Don't commit generated or machine-specific files.
- Don't remove the `skyfighter.*` storage keys or rename them. Doing so silently resets players' best scores and settings.
- Don't change `BASE_W`/`BASE_H` without checking the touch layout and HUD positioning.
