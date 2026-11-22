# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game (markup, CSS, and JavaScript) lives in one file, `index.html`. `README.md` is a placeholder and has no setup instructions.

There is no build step, package manager, test suite, or linter. Nothing is installed or compiled.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The whole game: inline `<style>`, the HUD and overlay markup, and one inline `<script>` that runs the game loop. |
| `README.md` | Placeholder title only. |

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No dependencies are needed.

## Architecture of `index.html`

- **Structure:** CSS in a `<style>` block near the top (`:root` color-scheme is dark). The `<canvas id="gameCanvas">` and the HUD/overlay DOM elements sit in the body. All logic is in a single `<script>` block that starts around line 422.
- **Rendering:** a 2D canvas driven by `requestAnimationFrame` through `loop(now)`, which calls `update(dt)` and then `draw()`. Drawing helpers include `drawPlane`, `drawBossShip`, and `drawClouds`. `resize()` handles fitting the canvas to the viewport.
- **Game state:** module-level `let` variables reset in `resetGame()`. These include `score`, `stage`, `stageKills`, `kills`, `frame`, and `spawnTimer`.
- **Stages and bosses:** each stage needs `KILLS_PER_STAGE` (18) kills before the boss spawns. `spawnBoss()` picks `BOSSES[(stage - 1) % BOSSES.length]`, and boss HP is `70 + stage * 35`. Visual themes come from `THEMES`, indexed the same way. Regular enemies are chosen from `ENEMY_TYPES`, filtered by each type's `minStage`.
- **Boss attacks:** the `fire*` functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) are patterns that `updateBoss` cycles through.
- **Difficulty:** the `DIFFICULTIES` table is selected through `setDifficulty()`.
- **Weapons and power-ups:** Vulcan and Laser weapons, each with three power levels, handled by `fireWeapon()` and `applyPowerUp()`. Smart bombs are triggered by `useBomb()`, and the player has a kill-chain score multiplier (`addScore`, `breakChain`).
- **Audio:** a Web Audio synth (`beep`, `noiseBurst`, and the `sfx` table). `ensureAudio()` creates the audio context on first user gesture.
- **Accessibility:** an `announce()` live region reports game events. A reduced-motion setting turns off screen shake and similar effects.
- **Input:**
  - Keyboard: arrows or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause. Keys are tracked in `keys[e.code]`.
  - Mouse: move to steer, click to fire. Handled through `canvasPointFromEvent`.
  - Touch: a virtual joystick (`joystickStart`/`joystickMove`/`joystickEnd`) plus on-screen FIRE and BOMB buttons.
- **Persistence:** `localStorage` keys, all prefixed `skyfighter.`:
  - `skyfighter.bestScore`
  - `skyfighter.muted`
  - `skyfighter.difficulty`
  - `skyfighter.reducedMotion`

  Every access should keep working when storage is unavailable (for example, private browsing). Keep any new storage key under the same prefix.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a restructure. Do not add a bundler, framework, or external CDN dependency without asking.
- Match the existing style: 2-space indentation, semicolons, `const`/`let`, and plain functions.
- Put tuning values (counts, HP curves, timing) in named constants near the existing ones rather than inlining magic numbers.
- Game-facing text lives in the DOM or in `announce()` calls. Keep it short and in the existing voice.
- Keep the layout working on phone-sized viewports. The HUD wraps, and touch controls must stay usable.

## Verification

With no test suite, verify changes by loading `index.html` in a browser and playing through the affected path: start a game, fire, take a hit, use a bomb, reach a boss, pause and resume, and check game over. For changes to layout or HUD, check at a narrow phone width as well as desktop. Use the browser console to catch errors.

## Git

- Development happens on the branch named in the session instructions. Do not push to other branches without permission.
- Only create a pull request when the user explicitly asks for one.
- Commit messages should describe the gameplay or code change in the imperative mood.
