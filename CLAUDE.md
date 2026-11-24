# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo holds **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html` (about 1,670 lines: inline `<style>` at lines ~10–359, markup at ~361–421, and one inline `<script>` at ~422–1667). `README.md` is a placeholder and has no project docs.

There is no build step, no package manager, no dependencies, no test suite, and no linter. Do not add any of these unless the user asks.

## Running the game

- Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`).
- For a quick check after a change, load the page, click Start, and play a few seconds. Use the browser tools (or Playwright, if available) to verify behavior. Screenshots are the best evidence for visual changes.

## Code layout inside `index.html`

- **Styles**: HUD, overlay, touch controls, and the vignette/flash effects are all in the `<style>` block. Colors and layout are inline CSS; there are no external stylesheets.
- **Markup**: canvas (`#gameCanvas`), HUD (`#hud`, with `#hearts`, `#bombs`, `#weaponStat`, `#multStat`), the start/pause overlay, and mobile touch controls (joystick zone, FIRE and BOMB buttons).
- **Script**: a single IIFE. Sections, roughly in order:
  - Config constants: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is synthesized with the Web Audio API; there are no audio files.
  - Input and layout: keyboard handlers, joystick and mouse/touch handling, `resize()` (base logical size is 480×720).
  - Game state and loop: `resetGame`, `update(dt)`, `draw()`, `loop(now)`. The `loop` function is driven by `requestAnimationFrame`.
  - Weapons and power-ups: `fireWeapon`, `applyPowerUp`. Weapon types are `vulcan` and `laser`; each has levels 1–3.
  - Bosses: `BOSS_PATTERNS` (bullet patterns), `updateBoss`, `killBoss`.
  - Persistence: `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.

## Controls (keep the docs in sync when changing these)

- Keyboard: arrows/WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses.
- Mouse: move to steer, click to fire. Touch: drag left side to fly, tap FIRE, tap BOMB.
- The on-screen control hint lives in `#controlsHint`. Update it whenever a binding changes.

## Gameplay tuning

- Difficulty values (lives, bombs, spawn rate, enemy speed, shoot chance, boss HP multiplier) are in `DIFFICULTIES`.
- Enemy stats are in `ENEMY_TYPES`; boss stats and names are in `BOSSES`.
- Stage pacing is `KILLS_PER_STAGE`. Stage themes cycle through `THEMES`.
- Prefer changing these constants over hard-coding numbers in logic.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Vanilla JS, no frameworks, no modules.
- Match the existing style: 2-space indent, `const`/`let`, arrow functions for handlers, short helper functions.
- Keep the game loop allocation-light. Reuse the existing `quality` / `trackPerf` mechanism for low-end devices instead of adding unconditional per-frame work.
- Respect the `reducedMotion` setting when adding shake, flash, or particle effects.
- Preserve mobile behavior: the layout must work at phone width, and `touch-action` / `user-select` stay disabled on the game.
- Keep `skyfighter.*` storage keys stable; changing them would reset players' saved best score, mute, and difficulty.

## Git

- Development branch for this work: `claude/gifted-tesla-utex5i`. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
