# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (one inline `<script>` wrapping a single IIFE, lines 422–1667).
- `README.md` — minimal project readme.

There is no build step, package manager, bundler, test suite, or external asset. Do not add one unless asked. Audio is synthesized with WebAudio, and sprites are drawn with canvas primitives.

To run the game, open `index.html` in a browser (or serve the directory with any static server). Nothing needs to be installed.

## Architecture (inside `index.html`)

The script is organized as `const`/`let` declarations and functions inside one IIFE, with `"use strict"`. Key sections, roughly in file order:

- **Config tables** (~lines 463–516): `STORAGE_*` localStorage keys, `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (one sky palette per stage, cycling), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six boss designs).
- **Persistence**: best score, mute, difficulty, and reduced-motion are stored in `localStorage` under the `skyfighter.*` keys. Every read and write must tolerate storage being unavailable.
- **Audio** (~lines 542–680): lazy `AudioContext` created by `ensureAudio()` on first user gesture; `beep()` and `noiseBurst()` are the synth primitives.
- **Canvas sizing** (~line 753): fixed logical resolution `BASE_W = 480`, `BASE_H = 720`, scaled to fit the viewport in `resize()`.
- **Input** (~lines 767–862): keyboard (arrows/WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause), a touch joystick and fire/bomb buttons on touch devices, and mouse/pointer aiming via `canvasPointFromEvent()`.
- **Game state** (~lines 863–912): module-level variables for score, lives, bombs, stage, kills, combo chain, and multiplier. `resetGame()` restores them.
- **Entities and spawning**: `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()`.
- **Weapons and bombs**: `fireWeapon()` (Vulcan/Laser with three power levels), `useBomb()` (screen-clearing, limited count), `applyPowerUp()`.
- **Bosses**: `BOSS_PATTERNS` holds the bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, and others); `updateBoss()` selects and runs them; `killBoss()` advances the stage.
- **Update / draw / loop**: `update(dt)` runs game logic, `draw()` renders everything, and `loop(now)` drives both via `requestAnimationFrame`. `trackPerf()` adapts render quality (`quality` 0/1/2) when frame times slip.
- **Feedback**: HUD (`renderHearts`, `renderBombs`), `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, and an `announce()` live region for screen readers.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Don't introduce frameworks, npm dependencies, or CDN scripts.
- Keep the code style already in the file: 2-space indentation, single quotes in JS, `const` for tuning tables, and short section banners of the form `// ---------- Name ----------`.
- Tuning values live in the config tables near the top of the script. Prefer changing those over hard-coding magic numbers in logic.
- Game-loop code uses `dt` in seconds. Scale movement and timers by `dt` rather than assuming a fixed frame rate.
- Anything new that touches the DOM, audio, or storage should respect the existing `reducedMotion`, `muted`, and try/catch patterns.
- Preserve mobile support: the layout uses `viewport-fit=cover`, safe-area insets, and `touch-action: none`. Test any HUD or control change at phone width.

## Working on the game

- Verify changes by loading `index.html` in a browser and playing through at least one stage. Check the touch path as well as keyboard when UI or input code changes.
- JavaScript syntax can be checked without a build by extracting the inline script, for example with `node --check` on the extracted file.
- When balancing, edit `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `BOSS_PATTERNS` rather than scattering constants through `update()`.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-q0iyd6`). Do not push to other branches without permission.
- Commit with clear, descriptive messages. Do not open a pull request unless the user asks for one.
