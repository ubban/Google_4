# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- `index.html` is the entire application: markup, CSS in `<style>`, and JavaScript in one inline `<script>`.
- `README.md` is a placeholder and has no setup steps.
- There is no build system, package manager, test suite, or linter. No dependencies are loaded from a CDN.

To run the game, open `index.html` in a browser. Nothing needs to be installed.

## Layout of `index.html`

- **Lines ~10–360 (`<style>`)**: all styling. The HUD is pill-shaped with heart and bomb icons, the overlay is a glassmorphic card, and touch controls are a joystick plus fire and bomb buttons.
- **Lines ~361–421 (markup)**: `#container` holds the `#gameCanvas` canvas, the HUD (`#hud`), the boss bar (`#bossBarWrap`), combo and stage banners, touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **Lines ~422–1667 (`<script>`)**: a single IIFE wrapping all game logic. It is `'use strict'`, and nothing is exposed on `window`.

Main sections inside the script, roughly in order:

| Section | Purpose |
| --- | --- |
| Constants (`DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`) | Tuning data. Edit these to rebalance the game. |
| Adaptive performance (`trackPerf`) | Steps a `quality` level (0–2) from frame timing. |
| Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`) | Web Audio synthesis. No audio files. |
| Input | Keyboard, joystick and buttons on touch, and mouse aim on the canvas. |
| `resetGame`, `update(dt)`, `draw()` | Game state reset, the per-frame simulation, and rendering. |
| Weapons and bombs (`fireWeapon`, `applyPowerUp`, `useBomb`) | Vulcan and Laser weapons, 3 power levels each, plus smart bombs. |
| Bosses (`BOSS_PATTERNS`, `updateBoss`, `killBoss`) | Multi-pattern boss fights with a health bar. |
| `loop(now)` | `requestAnimationFrame` loop. `dt` is clamped to 0.05 s. |

## Controls

- Move: arrow keys or WASD, or drag the mouse on the canvas, or the joystick on touch.
- Fire: Space (held), or hold the fire button or mouse button.
- Bomb: `X`, `Shift`, or the bomb button. Bombs clear the screen and damage bosses.
- Pause: `P` or `Escape`, or the pause button.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore`: best score, saved when a game ends.
- `skyfighter.muted`: `'1'` or `'0'`.
- `skyfighter.difficulty`: `easy`, `normal` or `hard`.
- `skyfighter.reducedMotion`: `'1'` or `'0'`. If unset, it follows `prefers-reduced-motion`.

Every `localStorage` access is currently unwrapped. If you add more, consider wrapping them in try/catch, because storage can throw in private windows or with blocked site data.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file layout is deliberate.
- Match the existing style: 2-space indentation, `const` and `let`, arrow functions for callbacks, and short section comments with `// ---------- Name ----------`.
- Keep new game data in the constant tables at the top of the script (for example, add a boss to `BOSSES` and its attack to `BOSS_PATTERNS`) rather than hard-coding values in logic.
- Keep the game's gameplay numbers in the difficulty table. `DIFFICULTIES` controls spawn rate, enemy speed, enemy fire chance, lives, bombs, and boss HP multiplier.
- Respect `reducedMotion`. Screen shake and flash effects should be gated on it.
- Keep the layout working at phone width. Touch controls are part of the game, not an add-on.

## Verifying changes

There are no automated tests. For changes to gameplay or rendering:

1. Open `index.html` in a browser, or serve the directory with any static server.
2. Play through at least one stage to a boss on the affected difficulty.
3. Check the console for errors, and check both keyboard and touch input if the change touches controls.
4. Check the layout at a narrow (phone) viewport.

If you add a test setup or build step, document the command here.

## Git

- Default branch work happens on the feature branch named in the session. Do not push to other branches without permission.
- Write commit messages in the imperative mood, with a short summary line and a body that explains the why for larger changes. The existing history uses this style, for example "Add Sky Fighter: Raiden-style vertical shoot-em-up game".
