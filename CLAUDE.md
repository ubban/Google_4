# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (~1,670 lines): markup, CSS, and one inline `<script>` block. There is no build step, no bundler, no package manager, and no dependencies.

Files:

- `index.html`: the whole game (HTML, `<style>`, and the inline game script).
- `README.md`: a one-line title. Keep it short.
- `CLAUDE.md`: this file.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- **Build:** none. Editing `index.html` is the build.
- **Tests:** none exist. Verify changes by playing the game in a browser and checking the console for errors. For behavior changes, exercise the affected path in each difficulty and on a narrow viewport when the change touches layout or touch controls.
- **Lint/format:** none configured. Match the existing style (see below).

## Code layout inside `index.html`

The inline script is one IIFE with `'use strict'`. Sections, in source order:

1. **DOM references** (`const xxxEl = document.getElementById(...)`). Every element the script touches is grabbed once at the top.
2. **Storage keys** (`STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`). Persisted to `localStorage` under the `skyfighter.*` prefix.
3. **Feedback helpers**: `announce` (screen-reader live region), `trackPerf` (adaptive quality from frame times).
4. **Audio**: a Web Audio synth. `beep` and `noiseBurst` build sounds, and the `sfx` table maps event names to sounds. No audio files are loaded. `ensureAudio` must run from a user gesture before any sound plays.
5. **Visual effects and HUD**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
6. **Settings**: mute, reduced motion, and difficulty (`setDifficulty`). The difficulty buttons are locked while a run is active.
7. **Canvas sizing**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the window, capped at 1.6x. Internal resolution never changes.
8. **Input**: keyboard (`keys` map plus `keydown`/`keyup`), touch joystick and buttons, and mouse steering via `canvas` events.
9. **Game data tables**: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, and `KILLS_PER_STAGE`.
10. **Game state and logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
11. **Boss attack patterns**: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`. `BOSS_PATTERNS[i]` is the pattern list for `BOSSES[i]`.
12. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
13. **Main loop**: `loop(now)` uses `requestAnimationFrame`. It clamps `dt` to 0.05 s and calls `update` then `draw` only while running and not paused.

## Game rules and tuning

Changing these values changes game balance, so do it deliberately:

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, then a boss spawns after a short delay. Beating the boss advances the stage. The stage's boss and theme cycle with `(stage - 1) % length`, so there are 6 bosses and 6 themes that repeat.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2).
- **Power-ups**: `life`, `bomb`, and `weapon`. Weapons are `vulcan` and `laser`, each with a level.
- **Score**: kills add points through `addScore`, scaled by a chain multiplier that `breakChain` resets. The best score persists in `localStorage`.
- **Controls**: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause. Mouse and touch controls are also supported.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a build step, a framework, or a dependency without asking.
- **Style.** Two-space indent, single quotes in JS, `const` by default, arrow functions for callbacks, and short section banners in the form `// ---------- Name ----------`. Match the surrounding code rather than reformatting it.
- **Naming.** `camelCase` for functions and variables, `UPPER_SNAKE` for constants and storage keys, and `sfx.<name>` for sound effects.
- **Frame-rate independence.** Movement and timers multiply by `dt` (seconds). Some older code uses `dt * 60` to scale per-frame speeds; follow the local pattern in the function you edit.
- **Accessibility.** Respect `reducedMotion`: shake and flash effects must check it. Announce important state changes through `announce()`.
- **Persistence.** Read and write `localStorage` only through the `STORAGE_*` keys.
- **Audio.** Create sounds through `beep` and `noiseBurst` rather than loading files. Route them through `masterGain` so mute works.
- **Responsive layout.** The UI must work on phones: touch controls appear for touch devices, and the canvas must scale without page scroll.

## Git workflow

- Develop on the branch named by the session (currently `claude/gifted-tesla-42plts`). Do not push to other branches without permission.
- Use clear, descriptive commit messages that explain why the change was made.
- Do not open a pull request unless the user asks for one.
