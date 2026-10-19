# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file, `index.html` (CSS, markup, and JavaScript inline). `README.md` is a placeholder.

There is no build step, no package manager, no framework, and no test suite. Do not add one unless asked.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Nothing needs to be installed.

## Repository layout

| Path         | Purpose                                                        |
| ------------ | -------------------------------------------------------------- |
| `index.html` | The entire game: styles (~lines 10–359), markup (~361–421), script (~422–end) |
| `README.md`  | Placeholder text                                               |

## Structure of `index.html`

The script is one IIFE wrapped in `'use strict'`. Its sections, in order, are:

1. **DOM lookups and storage keys**: `document.getElementById(...)` handles, and `skyfighter.*` `localStorage` keys (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
2. **Adaptive performance**: `trackPerf` samples frame times and moves `quality` (0 full, 1 medium, 2 low) to keep the frame rate up.
3. **Tunable data tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn timing, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed before a boss warning.
   - `THEMES`: six sky palettes, one per stage, cycling with the stage number.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
   - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with colors and an `hpMul`.
4. **Audio**: a lazily created `AudioContext` (`ensureAudio`) and synthesized sound effects (`beep`, `noiseBurst`, and the `sfx` map). Nothing is loaded from disk.
5. **HUD and effects**: `renderHearts`, `renderBombs`, flash and shake helpers, combo pop, stage banner, and `announce` for screen readers (writes to `#srLive`).
6. **Canvas sizing**: the canvas is a fixed `480×720` logical grid, CSS-scaled to fit the window (capped at 1.6×). Game coordinates always use `canvas.width` and `canvas.height`.
7. **Input**: keyboard, touch joystick and buttons, and mouse steering on the canvas. Keyboard and joystick input take priority over mouse steering.
8. **Game state and drawing**: `resetGame()` initializes the state. `drawPlane`, `drawBossShip`, and the other draw functions render the scene.
9. **Spawning and combat**: `spawnEnemy`, `spawnBoss`, `fireWeapon`, enemy and boss bullet patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), and `updateBoss`.
10. **Update and loop**: `update(dt)` advances the simulation. `loop(now)` runs under `requestAnimationFrame`, clamps `dt` to 50 ms, and calls `update` and `draw` only when the game is running and not paused.

## Gameplay reference

- **Controls**
  - Keyboard: arrow keys or WASD to move, Space to fire, `X` or Shift to drop a bomb, `P` or Escape to pause.
  - Mouse: move over the canvas to steer, hold the button to fire.
  - Touch: left-side joystick, fire button, bomb button.
- **Weapons**: Vulcan and Laser. Each weapon has three levels. Collecting a weapon power-up raises the level. After level 3 it switches to the other weapon at level 1.
- **Power-ups** (`maybeSpawnPowerUp`): life, bomb, and weapon.
- **Stages**: after `KILLS_PER_STAGE` kills, a boss warning appears and the boss spawns. Defeating the boss advances the stage and changes the theme.

## Conventions

- Keep the game in one file unless the user asks for a split. Match the existing style: two-space indentation, single quotes in JavaScript, `const` and `let`, arrow functions for callbacks, and short section comments in the `// ---------- Name ----------` form.
- Tune balance through the data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) instead of hard-coding numbers in logic.
- Use `announce()` for any new player-facing status text that screen readers should hear.
- Keep any new `localStorage` key under the `skyfighter.` prefix, and read and write it through a `STORAGE_*` constant.
- Preserve `prefers-reduced-motion` behavior: the `reducedMotion` flag (also toggled by the motion button) turns off flash effects and the CSS animations and vignette listed under `body.reduced-motion`.
- Keep the layout working on touch devices and at small widths. The page uses `touch-action: none` and `user-select: none` on purpose.

## Verifying changes

There is no automated test suite or linter. Verify changes with:

1. **Syntax check** of the inline script. For example, extract the `<script>` block to a file in a scratch directory and run `node --check` on it.
2. **Manual play test** in a browser: start a game, check the HUD, fire, collect power-ups, use a bomb, reach a boss, and check game over. Test at least one touch-sized viewport if the change touches input or layout.

Report honestly in your summary which of these you actually ran.

## Git

- Develop on the branch the session names. Do not push to `main` without explicit permission.
- Write clear, descriptive commit messages.
