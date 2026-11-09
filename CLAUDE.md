# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-page HTML5 canvas game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up with weapons, smart bombs, stages, and bosses.

- `index.html` — the entire game: CSS in `<style>`, DOM overlay/HUD markup, and one IIFE `<script>` containing all game logic.
- `README.md` — minimal project stub.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server. Verify changes by playing them in a browser (desktop and a narrow/mobile viewport), since there are no automated tests.

## Code layout inside `index.html`

The script is wrapped in `(function () { ... })()` and organized roughly as:

1. **DOM refs** — `document.getElementById(...)` lookups at the top of the script.
2. **Constants** — `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
3. **Persistence & settings** — best score, mute, difficulty, and reduced-motion state, read from and written to `localStorage`.
4. **Audio** — lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a `sfx` table.
5. **Input** — `keydown`/`keyup` into a `keys[e.code]` map; mouse and touch handlers for the canvas, joystick, fire, and bomb buttons.
6. **Game state & lifecycle** — `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `togglePause`, game over.
7. **Update** — `update(dt)` for the player, enemies, bullets, and collisions; `updateBoss(dt)` for boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
8. **Render** — `draw()` plus helpers such as `drawPlane`, `drawBossShip`, `drawClouds`.
9. **Loop** — `loop(now)` driven by `requestAnimationFrame`, with a frame-time tracker (`trackPerf`).

When adding a feature, follow the existing pattern: put tunables in the constant tables above rather than hard-coding them in logic.

## Game design facts (keep these in sync with the code)

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy shoot chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, then a boss spawns after a short delay. Defeating a boss advances `stage`.
- **Bosses and themes** cycle through the `BOSSES` and `THEMES` arrays; theme index is `(stage - 1) % THEMES.length`.
- **Enemies**: `grunt` (stage 1+), `interceptor` (stage 1+, weaves), `tank` (stage 2+, 4 HP).
- **Scoring**: per-enemy base scores, plus a kill-chain multiplier shown in the HUD.
- **Persistence keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Changing these names breaks saved player data; treat them as a stable contract.

## Controls

- Keyboard: arrow keys or `WASD` move; `Space` fires (held); `X` or `Shift` drops a bomb; `P` or `Esc` toggles pause.
- Mouse: moving over the canvas moves the ship toward the pointer (crosshair shown while active); pressing on the canvas also fires while held.
- Touch: left-side joystick zone to move; on-screen fire and bomb buttons.

## Conventions

- **Accessibility**: keep the `aria-live` announcer (`announce()`) in sync with important state changes (stage start, boss defeat, game over). Respect `prefers-reduced-motion` and the in-game reduced-motion toggle when adding effects such as shake, flash, or pulse.
- **Mobile**: the layout must work at phone width. Keep `touch-action` and the safe-area insets intact.
- **Audio**: create the `AudioContext` only from a user gesture (via `ensureAudio`). Respect the mute setting.
- **Style**: match the existing vanilla JS style (no frameworks, `const`/`let`, small helper functions, comments only where logic is non-obvious).
- Keep everything in `index.html` unless the user asks for a split. Do not introduce external CDN dependencies.

## Git workflow

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-8lfa8l`). Do not push to `main` without explicit permission.
- Do not open a pull request unless the user asks for one.
