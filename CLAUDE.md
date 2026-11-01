# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game. The entire game (markup, CSS, and JavaScript) lives in `index.html`. `README.md` is a placeholder.

There is no build step, package manager, bundler, or test suite. To run the game, open `index.html` in a browser, or serve the directory with any static server.

## Repository layout

- `index.html` (~1,670 lines): the whole app.
  - `<style>` (around lines 10–360): CSS for the HUD, vignette, title/overlay card, touch controls, and animations.
  - `<body>`: the `#gameCanvas` element, HUD, overlays, and touch controls.
  - `<script>` (starts around line 422): one IIFE that holds all game state and logic. Nothing is exposed on `window`.
- `README.md`: placeholder.

## Architecture of the game script

The script is organized in sections, marked by `// ---------- Name ----------` comments:

- **Config constants:** `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`. Tune balance here rather than in logic.
- **Persistence:** `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write should tolerate storage being unavailable.
- **Adaptive performance:** `trackPerf` samples frame times and steps `quality` (0 full, 1 medium, 2 low) to keep the frame rate up.
- **Audio:** a lazily created `AudioContext` (`ensureAudio`) with `beep` and `noiseBurst` helpers. Audio starts only after a user gesture.
- **Input:** keyboard listeners on `window`, mouse/pointer steering on the canvas, and a virtual joystick plus FIRE and BOMB buttons for touch devices.
- **Sizing:** the logical play field is `BASE_W × BASE_H` (480 × 720). `resize()` scales the canvas to fit the viewport.
- **Game state:** module-level `let` variables in the script, including `stage`, `kills`, `stageKills`, `bossWarned`, and `frame`. `resetGame()` restores them.
- **Entities and spawning:** `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, and `spawnExplosion`.
- **Weapons and bombs:** `fireWeapon` (Vulcan and Laser, three power levels each) and `useBomb` (clears enemy bullets and enemies and damages bosses). Bombs come from a limited stock (`bombCount` / `maxBombs`, set by difficulty).
- **Boss patterns:** `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`, selected in `updateBoss`.
- **Update and render:** `update(dt)` runs game logic, `draw()` renders, and `loop(now)` drives both through `requestAnimationFrame`. Keep logic out of `draw()`.
- **Lifecycle:** `loseLife`, `killBoss`, `endGame`, and `togglePause`.

### Game rules worth knowing

- After `KILLS_PER_STAGE` (18) kills in a stage, a boss spawns. Defeating it advances the stage. Themes from `THEMES` cycle by stage number.
- `tank` enemies first appear in stage 2 (`minStage`).
- Difficulty sets lives, starting bombs, spawn rate, enemy fire rate, enemy speed, and boss HP multiplier.
- The kill-chain combo multiplies score. `breakChain()` resets it.

## Controls (for testing)

- Desktop: mouse to steer and click to fire. Arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Escape to pause.
- Touch: drag in the left zone to fly, tap FIRE, tap BOMB.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add a build system, framework, or dependencies without a clear reason.
- Match the existing style: 2-space indentation, single quotes, `const` by default, and section banner comments.
- Draw with the canvas 2D API only. Do not add image assets or external fetches. Visuals are procedural (`drawPlane`, `drawBossShip`, `drawClouds`).
- Respect `reducedMotion` for any new screen shake, flash, or animation. Respect `muted` for any new sound.
- Honor the adaptive `quality` level for expensive effects.
- Keep UI text and overlays accessible: use `announce()` for screen-reader updates, and keep `aria-live` regions intact.

## Verifying changes

There is no automated test suite or linter configured. Verify changes by:

1. Opening `index.html` in a browser (or serving it statically) and checking for console errors.
2. Playing through at least one stage, a boss fight, a life loss, and a game over on the affected difficulty.
3. Checking the mobile layout and touch controls when UI or input code changes.
4. Confirming that settings persist across reloads (mute, difficulty, reduced motion, best score).

## Git

- Development happens on feature branches. Do not push to `main` directly.
- Commit messages should describe the gameplay or code change, using the imperative mood.
