# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>` block, ~lines 10–362), canvas element, HUD/overlay markup, and all JavaScript (one IIFE in `<script>`, ~lines 422–1667).
- `README.md` — placeholder title only.

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. The game runs by opening `index.html` in a browser.

## Running and testing

- Run: open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- There are no automated tests or linters configured. Verify changes by playing the game in a browser: start a run, check the affected behavior, and check the browser console for errors.
- For mobile-related changes, test with touch emulation (drag-to-fly on the left, FIRE and BOMB buttons on the right).

## Architecture of `index.html`

All game logic lives inside one `(function () { 'use strict'; ... })()` closure. Key pieces, roughly in file order:

- **DOM refs** — cached at the top of the IIFE (`canvas`, HUD elements, overlay, touch controls, buttons).
- **Persistence** — `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. All reads/writes are direct; keep that prefix for any new keys.
- **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`. Prefer changing balance here rather than in logic.
- **Audio** — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX`). No audio files. Audio is created lazily on first user gesture.
- **Input** — keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and touch joystick + fire/bomb buttons. Pause is `P`/`Esc`; bomb is `X` or `Shift`; fire is `Space` or held mouse/touch.
- **Layout** — `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical play area to fit the viewport.
- **Game state** — `resetGame()` initializes state; `update(dt)` advances simulation each frame; `draw()` renders; `loop(now)` drives the `requestAnimationFrame` loop with a delta-time step.
- **Weapons and power-ups** — `fireWeapon()` (Vulcan/Laser, 3 levels), `applyPowerUp()`, `maybeSpawnPowerUp()`, `useBomb()` (clears the screen, damages bosses).
- **Bosses** — `spawnBoss()`, `updateBoss()`, `killBoss()`, and the `fire*` pattern helpers wired into `BOSS_PATTERNS`.
- **Lifecycle** — `loseLife()`, `endGame()`, `togglePause()`, `startBtn` click handler starts a run.
- **Accessibility** — `announce()` writes to the `srLive` live region; a reduced-motion toggle disables screen shake and similar effects.

## Conventions

- Keep the game as a **single self-contained HTML file** unless the user explicitly asks to split it. Do not introduce a build system, framework, or CDN dependency without asking.
- Stay in vanilla JS and the existing style: `const`/`let`, arrow functions in event handlers, 2-space indentation, single quotes.
- Everything stays inside the IIFE. Do not add new globals.
- Update the controls hint (`#controlsHint`) if you change or add controls.
- Respect the reduced-motion setting for any new screen-shake, flash, or animation effect.
- Keep touch parity: any new keyboard action needs a touch equivalent.

## Git workflow

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-mpo4ox`). Do not push to other branches without permission.
- Commit messages describe the gameplay or UI change in plain words, matching the style of existing commits.
- Do not create pull requests unless the user asks.
