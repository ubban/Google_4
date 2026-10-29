# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

- `index.html` is the entire game: inline CSS (`<style>`), markup, and one inline `<script>` block.
- `README.md` is a placeholder and has no setup instructions.
- There is no build step, package manager, test suite, linter, or CI configuration.

## Running the game

Open `index.html` directly in a browser. No server or install step is needed.

## Code layout inside `index.html`

- **CSS** (lines 10–359): HUD pills, hearts/bombs, glassmorphic overlay card, touch controls, and the reduced-motion and vignette styles.
- **Markup** (lines 361–421): the `#gameCanvas` canvas plus HUD, overlay, touch-control, and boss-bar elements, all referenced by ID.
- **Script** (lines 422–1667): a single IIFE with `'use strict'`. Sections, in order:
  - Element lookups and config constants (`DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `STORAGE_*`).
  - Audio via the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`).
  - HUD and effects helpers (`renderHearts`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`).
  - Input: keyboard handlers, a touch joystick, fire and bomb buttons, and mouse/canvas pointer handling.
  - Game state and mechanics: `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `update`, `loseLife`, `endGame`, `togglePause`.
  - Boss attack patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`).
  - Rendering (`drawPlane`, `drawBossShip`, `drawClouds`, `draw`).
  - Main loop (`loop`), driven by `requestAnimationFrame` with `dt` capped at 0.05 s.

## Controls

- **Keyboard:** arrow keys or WASD to move, Space to fire, `X` or Shift to use a bomb, `P` or Escape to pause.
- **Touch:** on-screen joystick for movement, fire button, and bomb button.
- **Mouse:** pointer movement and mouse-down on the canvas.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

| Key | Stores |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; defaults from user preference when unset |

Keep these keys stable, since changing them resets players' saved settings. If you add a new key, use the `skyfighter.` prefix and a matching `STORAGE_*` constant.

## Conventions

- Keep the game as a single self-contained file. Do not add a build step, bundler, or external dependencies unless the user asks.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, `camelCase` for functions and variables, and `UPPER_SNAKE_CASE` for tuning constants and storage keys.
- Put tuning values (speeds, spawn rates, difficulty multipliers, boss HP scaling) in the existing config objects rather than hard-coding them in mechanics.
- Respect the reduced-motion setting when adding visual effects such as shake, flashes, or pulses.
- Keep the 480×720 base logical resolution (`BASE_W`, `BASE_H`) in mind when changing layout or drawing code; `resize()` scales the canvas to the window.
- Preserve accessibility hooks: announce important state changes through the `srLive` live region (`announce()`).

## Verification

There are no automated tests. After changing game logic or rendering, open `index.html` in a browser and check:

1. Start, play, and pause and resume work, including via `P` and Escape.
2. Movement, firing, and bombs behave as expected, and bombs damage bosses.
3. Each difficulty loads with the correct lives and bomb counts.
4. Best score and settings persist across a page reload.
5. The touch controls work at phone width, or in a responsive device emulator.
6. The browser console shows no errors.

## Git workflow

- Develop on the branch the session designates. Do not push to `main` unless explicitly asked.
- Write clear, descriptive commit messages that explain the why as well as the what.
- Only open a pull request when explicitly asked.
