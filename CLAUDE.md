# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — a placeholder with no setup instructions.

There is no build step, no package manager (`package.json` does not exist), no bundler, no framework, and no automated test suite. Do not add tooling unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server. No install step is needed.

## Code layout inside `index.html`

- **`<style>`** (about lines 10–360): all styling. HUD, overlay card, touch controls, and the theme and animation rules live here.
- **`<body>`** (about lines 361–421): the DOM skeleton. Key IDs include `gameCanvas`, `hud`, `bossBarWrap`, `touchControls`, `overlay`/`overlayCard`, and the `diffBtn` difficulty buttons.
- **`<script>`** (about lines 422–1668): one IIFE wrapping the whole game, with `'use strict'`. Nothing is exposed on `window`. Keep new code inside that IIFE.

Major sections of the script, in order:

1. DOM lookups and `const` references to elements.
2. Storage keys (`skyfighter.*`) and persisted settings: best score, mute, difficulty, reduced motion.
3. Game data tables: `DIFFICULTIES`, `THEMES` (one per stage), `ENEMY_TYPES`, `BOSSES`.
4. Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is created lazily on the first user gesture.
5. Input: keyboard, the touch joystick and buttons, and mouse steering on the canvas.
6. Game state: `resetGame`, `update(dt)`, and the spawn, collision, and scoring helpers.
7. Weapons and bosses: `fireWeapon`, `useBomb`, `BOSS_PATTERNS` (one entry per boss, listing its attack patterns, which call helpers such as `fireRadial` and `fireCross`), `updateBoss`, `killBoss`.
8. Rendering: `draw`, `drawPlane`, `drawBossShip`, `drawClouds`.
9. Main loop: `loop(now)` drives everything via `requestAnimationFrame`. `dt` is clamped to 50 ms.

## Game design constants

These are the knobs to change when tuning gameplay:

- `DIFFICULTIES`: spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier for `easy`, `normal`, and `hard`.
- `KILLS_PER_STAGE` (18): kills needed to advance a stage and reach the boss.
- `THEMES`: the sky and cloud palette for each stage. The theme index cycles with the stage.
- `ENEMY_TYPES` and `BOSSES`: sizes, HP, speed, score values, and colors.

Keep balance changes in these tables rather than scattering magic numbers through the logic.

## Controls (keep in sync with the on-screen hint)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag on the left to fly, tap FIRE, tap BOMB.

The `#controlsHint` element in the HTML lists these controls. Update it whenever a binding changes.

## Persistence

`localStorage` is the only persistence, and every access uses a `skyfighter.`-prefixed key. Keep the existing `STORAGE_*` constants and use them for any new key. Reads should fall back to a default when the key is missing.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and short helper functions with descriptive names.
- Keep the single-file structure. Do not split out modules, a build step, or external dependencies unless the user asks.
- Do not add comments that only restate the code. Brief comments explaining why something is non-obvious are fine.
- Respect `reducedMotion` for any new screen-shake, flash, or motion effect. Check how the existing effects gate on it.
- Keep the game playable on touch devices. Any new control must work with the touch zones and the phone-width layout.

## Verifying changes

There is no test runner or linter. To check a change:

1. Open `index.html` in a browser (or a headless browser) and confirm there are no console errors on load.
2. Start a game, then exercise the path you changed: movement, firing, a bomb, a pause/resume, a stage transition, and a boss fight if relevant.
3. Check the touch layout at a phone-sized viewport if you touched the HUD, overlay, or controls.

A quick syntax check of the inline script is also useful. For example, extract the `<script>` block into a temporary file and run `node --check` on it.

## Git

- Develop on the branch assigned for the session, not on the default branch.
- Write clear, descriptive commit messages that explain the change.
