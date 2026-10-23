# CLAUDE.md

Guidance for working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-file HTML5 canvas game. The entire game (markup, CSS, and JavaScript) lives in `index.html`. `README.md` is a placeholder.

There is no build step, package manifest, test suite, or linter. To run the game, open `index.html` in a browser.

## Repository layout

- `index.html` — the whole game:
  - Lines ~10–359: `<style>` block (HUD, overlay card, touch controls, animations, reduced-motion styles).
  - Lines ~361–421: markup (canvas, HUD elements, overlay, touch controls, control hints).
  - Lines ~422–1667: one IIFE in a `<script>` tag containing all game logic.
- `README.md` — placeholder.

## Architecture of the game script

All game code sits inside `(function () { 'use strict'; ... })()`, so nothing leaks to `window`. Major sections, roughly in source order:

- **DOM refs and constants** — element lookups at the top, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
- **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should stay tolerant of missing values.
- **Audio** — a lazily created `AudioContext` (`ensureAudio`), with synthesized sound through `beep` and `noiseBurst`. Sound is gated by the mute flag.
- **Adaptive quality** — `trackPerf` samples frame times and steps `quality` (0 full / 1 medium / 2 low) to keep frame rate up.
- **Input** — keyboard state in `keys` (keyed by `e.code`), mouse and pointer steering on the canvas, a touch joystick plus FIRE/BOMB buttons, and a `firing` flag shared across inputs.
- **Game state and flow** — `resetGame`, `update(dt)`, `draw()`, `loop(now)` (driven by `requestAnimationFrame`, with `dt` clamped to 50 ms), `loseLife`, `endGame`, `togglePause`, `killBoss`.
- **Entities and rendering** — enemies (`spawnEnemy`), bosses (`spawnBoss`, `updateBoss`, `BOSS_PATTERNS`, `drawBossShip`), player plane (`drawPlane`), power-ups (`maybeSpawnPowerUp`, `applyPowerUp`), explosions, parallax clouds (`drawClouds`), and HUD helpers (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`).
- **Scoring** — kill-chain multiplier (`addScore`, `breakChain`) and stage progression every `KILLS_PER_STAGE` kills.

Keep new behavior in the matching section and follow the existing `// ---------- Section ----------` comment banners.

## Game rules and controls (keep these in sync with code)

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Weapons**: Vulcan and Laser, each with 3 power levels, switched by power-ups.
- **Bombs**: limited per run. They clear the screen and damage bosses.
- **Bosses**: `BOSSES` lists six, cycled by stage. Each has a health bar and a set of attack patterns.
- **Controls**: mouse moves and clicks to fire; Arrows or WASD move; Space fires; X or Shift bombs; P or Esc pauses. Touch uses a left-side drag joystick, a FIRE button, and a BOMB button.

## Conventions

- Plain browser JavaScript, ES2015+ syntax (`const`/`let`, arrow functions, template literals). No frameworks, no bundler, no external dependencies. Don't add a CDN or npm dependency unless the user asks for one.
- Keep everything in `index.html` unless the user asks for a split. A split would need matching changes to how the file loads.
- Match the existing style: 2-space indentation, single quotes in JS, section banners, and short comments that explain *why* rather than *what*.
- Game constants belong in the constant objects near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`), not hard-coded in update or draw code.
- Accessibility features to preserve: the `srLive` live region for announcements (`announce`), the `prefers-reduced-motion` setting (which toggles the `reduced-motion` body class, hides flash and vignette effects via CSS, and softens `screenShake`), and the mute toggle.
- Canvas logical size is `BASE_W` × `BASE_H` (480 × 720), scaled by `resize()`. Keep game coordinates in that space.

## Verifying changes

There are no automated tests or lint commands. To verify a change:

1. Open `index.html` in a browser (or run a simple static server) and check the browser console for errors.
2. Play through at least one stage, a boss fight, a pause/resume, and a game over, on each difficulty you touched.
3. Check touch controls with device emulation when the change touches input or layout.
4. Confirm that the best score, mute, difficulty, and reduced-motion settings persist across reloads.

If you add a test or build tool, document the command here.

## Git

- Development branch for this work: `claude/gifted-tesla-xvnawe`.
- Commit messages: a short imperative subject line, then a body that explains the change when it isn't obvious from the diff.
- Don't open a pull request unless the user asks for one.
