# CLAUDE.md

Guidance for working in this repository.

## Project overview

This is **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-page HTML5 canvas game. The entire game (markup, CSS, and JavaScript) lives in `index.html`. `README.md` is a placeholder and has no project documentation.

There is no build system, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser. No server or install step is needed.

## Code layout (`index.html`)

- **`<style>`** (around lines 10–360): all CSS. The HUD, overlay card, touch controls, and reduced-motion styles are here. Body classes such as `.reduced-motion` drive some visual effects.
- **Markup** (around lines 362–420): `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`), boss bar, combo and stage banners, top-bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls, and the start/game-over `#overlay`.
- **`<script>`** (starts around line 422): one IIFE wrapping all game logic. Nothing is exposed on `window`. Keep new code inside the IIFE.

Major sections of the script, in order:

1. **Config constants**: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES`.
2. **Persistence and accessibility**: best score, mute, difficulty, and reduced motion are stored in `localStorage` under `skyfighter.*` keys. Reduced motion defaults to the `prefers-reduced-motion` media query until the user toggles it.
3. **Audio**: a Web Audio API synth (`ensureAudio`, `beep`, `noiseBurst`). Sound effects are generated, not loaded from files.
4. **Input**: keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and touch joystick plus FIRE and BOMB buttons.
5. **Game state**: `resetGame()` builds `player` and the other entity arrays.
6. **Update and draw**: `update(dt)` handles simulation. `draw*` functions render. The main `loop` runs on `requestAnimationFrame`.
7. **Weapons and bosses**: `fireWeapon` (vulcan and laser, each with power levels), `useBomb`, `applyPowerUp`, and `BOSS_PATTERNS` with per-pattern fire helpers (`fireRadial`, `fireAimedSpread`, `fireHomingSeeded`, and others). `updateBoss` drives boss phases.

## Gameplay rules to preserve

- Controls: mouse steers and clicks to fire, Arrows or WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses. Touch uses a left drag zone, FIRE, and BOMB.
- Difficulty (`easy`, `normal`, `hard`) sets spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- Stages advance every `KILLS_PER_STAGE` kills and end in a boss fight. Stage themes cycle through `THEMES`.
- Score uses a kill-chain multiplier that `breakChain()` resets.
- Power-up types are `life`, `bomb`, and `weapon`.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, and short section headers in the form `// ---------- Name ----------`.
- Keep everything in `index.html` unless there is a clear reason to split it out. If you do split files, update this document.
- Use `dt`-based movement so physics stays frame-rate independent.
- Respect reduced-motion. Screen shake, flashes, and pulses should check `reducedMotion`.
- Keep the layout working on mobile (`touch-action: none`, safe-area insets, and the `resize()` scaling to the 480×720 base canvas).
- Announce important state changes through the `announce()` live region for screen readers.

## Verifying changes

No automated tests exist. To verify a change:

1. Open `index.html` in a browser and check the console for errors.
2. Play through at least one stage, a boss fight, and a game over.
3. Check the touch layout using browser device emulation.
4. Confirm that mute, difficulty, and reduced-motion settings persist across reloads.

## Git

- Development happens on feature branches. Do not push directly to the default branch.
- Write commit messages that describe the gameplay or code change.
