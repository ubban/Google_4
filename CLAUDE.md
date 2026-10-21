# CLAUDE.md

Guidance for Claude Code working in this repository.

## What this is

**Sky Fighter** is a Raiden-style vertical shoot-em-up that runs in the browser. The whole game is one static file, `index.html`. There is no build step, package manager, bundler, framework, or test suite.

`README.md` contains only a timestamp line and is not real documentation.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server. Nothing needs to be installed.

## File layout

`index.html` (~1670 lines) has three sections:

| Lines (approx.) | Section | Contents |
|---|---|---|
| 1–361 | `<style>` | All CSS. HUD, overlay card, touch controls, boss bar, flash/vignette effects. |
| 362–421 | `<body>` markup | `#container` with `#gameCanvas`, the HUD (`#hud`), boss bar, touch controls (`#touchControls`), and the start/pause overlay (`#overlay`). |
| 422–1667 | `<script>` | One `(function () { 'use strict'; ... })();` IIFE holding all game logic. |

Nothing is exposed on `window`. Keep new state and helpers inside the IIFE.

## Architecture of the script

The script is organized into `// ---------- Section ----------` blocks. Roughly in order:

1. **DOM refs and constants**: element lookups, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
2. **Persistence**: best score, mute, difficulty, and reduced-motion are stored in `localStorage` under `skyfighter.*`.
3. **Adaptive performance**: `trackPerf` moves a `quality` level (0 full, 1 medium, 2 low) based on a 45-frame rolling average.
4. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` primitives and a `sfx` object of named sounds.
5. **HUD and feedback**: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, `flashBomb`, `announce` (aria-live).
6. **Resize**: the canvas is sized from a logical `BASE_W` × `BASE_H` of 480 × 720.
7. **Input**: keyboard (`keys` map), touch joystick and buttons, and mouse steering and firing.
8. **Game state**: `resetGame()` initializes `player`, `lives`, `bombCount`, score, and stage.
9. **Entities and spawning**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`.
10. **Combat**: `fireWeapon` (vulcan and laser, levels 1–3), `useBomb`, `addScore` with a combo multiplier (`breakChain`), `applyPowerUp`.
11. **Boss AI**: `BOSS_PATTERNS` is a list of attack functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss` picks from them and `killBoss` advances the stage.
12. **Update and draw**: `update(dt)` advances the simulation. `draw()` renders everything, including `drawPlane`, `drawBossShip`, and `drawClouds`.
13. **Loop and lifecycle**: `loop(now)` drives `requestAnimationFrame`. `dt` is clamped to 50 ms. `togglePause`, `loseLife`, and `endGame` handle the state transitions.

### Core rules

- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills. A boss then spawns after a 1.2 s delay. Defeating it increments `stage`. The background theme is `THEMES[(stage - 1) % THEMES.length]`, so the six themes cycle.
- **Lives and hits**: the player has `invuln` frames for 1.5 s after a hit, and the sprite blinks while invulnerable.
- **Bombs**: `useBomb` removes all enemies and enemy bullets, awards small points per enemy, and deals 24 damage to the boss. The starting count depends on difficulty.
- **Power-ups**: `life`, `bomb`, and `weapon`. Weapon upgrades go level 1 → 2 → 3, then toggle between vulcan and laser.
- **Difficulty** (`DIFFICULTIES`) changes spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Best score** is saved to `localStorage` only when a game ends with a new high score.

## Controls

- **Keyboard**: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- **Mouse**: move to steer, click to fire.
- **Touch**: drag on the left half to fly, tap FIRE, tap BOMB.

## Conventions

Match the existing code:

- Vanilla JavaScript (ES2015+) and inline CSS. Don't add dependencies, a build step, or external CDN scripts unless the user asks.
- 2-space indentation, single quotes in JS, semicolons, `const` by default, `let` for mutable state.
- Group code under `// ---------- Name ----------` headers. Use short `//` comments for non-obvious logic only.
- Name tuning values as named constants or entries in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) instead of inline magic numbers.
- Keep `'use strict'` and the IIFE wrapper.
- Persist settings through the `STORAGE_*` keys and wrap new storage use in the same pattern.
- Honor reduced motion: anything that shakes, flashes, or pulses should check `reducedMotion`.
- Announce meaningful state changes through `announce()` so screen readers get them.
- Keep the layout working on touch devices and at small widths. The overlay and touch controls are part of the core experience.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser, or run `python3 -m http.server` in the repo and visit it.
2. Check the browser console for errors on load and during play.
3. Exercise the affected path: start a game, play to a boss for boss changes, lose all lives for game-over changes, toggle mute, pause, reduced motion, and difficulty.
4. For layout or input changes, test a narrow viewport or device emulation with touch input.

A syntax-only check of the script is also useful. For example, extract the `<script>` body and run `node --check` on it.

## Git

- Development branch for this work: `claude/gifted-tesla-pl9wvr`. Base the work on `main`.
- Commit messages are short imperative summaries, matching the existing history (for example, "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
- Don't open pull requests unless the user asks.
