# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up written as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–360), and JavaScript (`<script>`, ~lines 422–end). |
| `README.md` | Placeholder (title only). |

There is no build system, package manager, bundler, test suite, linter, or CI config. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, covering the paths you touched (see the checklist below).
- Use a headless browser (for example Playwright) for scripted checks when useful. Keyboard, mouse, and touch controls all go through the same code paths, so test the input method relevant to your change.

## Code architecture (`index.html`)

The script is one IIFE with `'use strict'`. Everything lives in closure scope; nothing is exported to `window`. Keep it that way unless there is a clear reason.

Major sections, in order of appearance:

1. **DOM refs**: `const xxxEl = document.getElementById(...)` at the top. Add new HUD or overlay elements here.
2. **Storage keys**: `STORAGE_*` constants for `localStorage` (best score, mute, difficulty, reduced motion). Keys are namespaced `skyfighter.*`.
3. **Adaptive quality**: `trackPerf()` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to keep the frame rate up.
4. **Tuning tables**:
   - `DIFFICULTIES`: spawn rate, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
   - `THEMES`: six sky palettes that cycle by stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). Each has a `minStage` gate.
   - `BOSSES`: six bosses, each with colors and an `hpMul`. Attack patterns are in `BOSS_PATTERNS`.
5. **Audio**: `ensureAudio()` creates the `AudioContext` lazily, on a user gesture. `beep()` and `noiseBurst()` are the sound primitives. `EXPLODE_SFX` maps enemy types to sounds.
6. **UI helpers**: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (for screen readers via the `srLive` live region).
7. **Input**: keyboard (`keydown`/`keyup` into a `keys` map keyed by `e.code`), mouse (move to steer, click to fire), and touch (virtual joystick plus FIRE and BOMB buttons).
8. **Game state and logic**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
9. **Rendering**: `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`. Drawing is canvas 2D only.
10. **Loop**: `loop(now)` clamps `dt` to 50 ms, updates and draws only when `running && !paused`, and reschedules via `requestAnimationFrame`.

### Conventions used in this file

- Game logic uses a fixed base resolution, `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to the viewport. Coordinates are in base units, not CSS pixels.
- Time-based motion multiplies by `dt` (seconds). Do not use per-frame increments.
- Entities are plain objects in arrays. Collision uses `rectsOverlap`.
- Persistent settings are written to `localStorage` through a `set*` function (`setDifficulty`, mute toggle, motion toggle). Read them at startup. Wrap storage access with care, since it can throw in restricted browsers.
- Accessibility: respect `prefers-reduced-motion` (`reducedMotion`) when adding screen shake, flashes, or other motion effects. Announce important state changes with `announce()`.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and short section comments with `// ---------- Name ----------`.

## Making changes

- **Keep the single-file design.** Do not split the game into modules or add a build step unless the user asks for one.
- **Balance changes** belong in the tuning tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`) rather than hard-coded in logic.
- **New enemies, bosses, or power-ups**: add the data entry, then the spawn or apply logic, then the rendering branch in `draw()`. Check the `minStage` gating.
- **Visual changes**: keep the existing canvas drawing style (shaded planes, glow, parallax clouds) unless asked to redesign.
- **Touch and mouse**: any new control must work on desktop and on touch devices. The touch layout is in `#touchControls`.

## Manual test checklist

After any gameplay change, verify in a browser:

- [ ] Start, play, and die: lives decrement, the game-over overlay shows the stats, and the best score persists after reload.
- [ ] Pause with `P` or `Esc`, and resume with the Start button.
- [ ] Each difficulty works, and the choice persists after reload.
- [ ] Bomb with `X` or `Shift` clears the screen and uses up a bomb.
- [ ] Clear a stage to reach a boss, and check the boss health bar and that the boss can be defeated.
- [ ] Power-ups (life, bomb, weapon) are collected and applied.
- [ ] Mute and reduced-motion toggles work and persist.
- [ ] No console errors. Check the console on any change.
- [ ] The layout still works at a phone-sized viewport and with touch controls.

## Git

- Develop on the feature branch named in the session instructions. Do not push to `main` directly.
- Write commit messages in the imperative mood with a short summary line. Explain the why in the body when it is not obvious.
- Do not open a pull request unless the user asks for one.
