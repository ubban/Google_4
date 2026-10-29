# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (~360 lines), CSS in a `<style>` block, and vanilla JS in one IIFE `<script>` block. |
| `README.md` | Placeholder title only. |

There is no build step, no package manager, no test suite, and no linter configuration. Nothing needs installing.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`). Both work because the game uses no modules or fetches.
- **Test:** there is no automated test suite. Verify changes by playing the game in a browser, on both desktop (keyboard/mouse) and a touch-sized viewport, since mobile controls are part of the game.

## Architecture of `index.html`

Everything lives in one IIFE (`(function () { 'use strict'; ... })();`). Helpers and state are closures inside it; nothing is exported to `window`.

Major sections, in file order:

1. **DOM references** (`const xxxEl = document.getElementById(...)`) at the top of the script. Add new element lookups here.
2. **Storage keys** (`STORAGE_*`, prefixed `skyfighter.`): best score, mute, difficulty, reduced motion.
3. **Adaptive quality** (`trackPerf`, `quality` 0/1/2): samples frame times and lowers visual detail when the frame rate drops.
4. **Tuning tables**
   - `DIFFICULTIES`: spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed before a boss warning.
   - `THEMES`: six sky palettes, cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (4 HP, unlocks at stage 2).
   - `BOSSES`: six bosses, one per stage, with names, colors and HP multipliers.
5. **Audio**: `ensureAudio()` lazily creates the `AudioContext`. `beep()` and `noiseBurst()` synthesize all sound effects; there are no audio files. Mute state persists.
6. **HUD helpers**: hearts, bombs, combo popups, stage banner, screen shake, hit and bomb flashes, and `announce()` for screen readers.
7. **Layout and input**: `resize()` scales a fixed `BASE_W` × `BASE_H` (480 × 720) logical canvas to the window. Keyboard, mouse, and touch (joystick and buttons) handlers are registered together near the bottom of the setup block.
8. **Game state**: `resetGame()` initializes the state variables. `update(dt)` runs game logic, and `draw()` renders. Both are driven by the `loop(now)` requestAnimationFrame loop, where `dt` is capped at 50 ms.
9. **Weapons and power-ups**: `fireWeapon()` with Vulcan/Laser at 3 levels, `applyPowerUp()` (weapon level, bomb, extra life), and `useBomb()`, which clears the screen and damages bosses.
10. **Bosses**: `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss()` cycles through them, and `killBoss()` advances the stage.
11. **Flow**: `loseLife()`, `endGame()`, and `togglePause()`.

Scoring includes a kill-chain multiplier (`addScore`, `breakChain`).

## Conventions

- **Keep it in one file.** The game is deliberately a single self-contained `index.html`. Do not split it into modules, add a bundler, or add dependencies unless the user asks.
- **Tune with the tables.** Balance changes (speeds, spawn rates, HP, boss stats) should go in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES`, not hard-coded in logic.
- **Keep the code style.** Use the surrounding idioms: `const`/`let`, arrow callbacks, section comments with `// ---------- Name ----------`, and short helper functions. Match existing naming (`spawnX`, `fireX`, `drawX`, `updateX`).
- **Respect the frame loop.** Game logic belongs in `update(dt)` and rendering in `draw()`. Use `dt` for movement and timers so behavior stays consistent across frame rates.
- **Persist with the existing keys.** Store settings with the `skyfighter.*` `localStorage` keys. Current `localStorage` calls are not wrapped in `try/catch`; if you add new storage access, consider guarding it so the game still runs when storage is blocked.
- **Accessibility and motion.** Honor `prefers-reduced-motion` (the game reads it on load and has a toggle). New screen-shake or flashing effects should go through the reduced-motion path.
- **Keep both input methods working.** Any new control must work with the keyboard (`e.code`), the mouse, and touch. Update the on-screen `controlsHint` text when controls change.
- **Audio is synthesized.** Add sound effects with `beep()` or `noiseBurst()` rather than adding audio files.

## Controls (for reference when changing input)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- Touch: drag in the left zone to fly (joystick), tap FIRE, tap BOMB.

## Git and workflow

- Develop on the branch the session specifies, and do not push to other branches without permission.
- Commit with clear, descriptive messages that explain the gameplay or code change.
- Do not open a pull request unless the user asks for one.
