# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file, `index.html`, with inline CSS and one inline `<script>` block. There is no build step, no package manager, and no external dependencies.

Files:

- `index.html` (~1,670 lines): the complete game (markup, styles, and logic).
- `README.md`: a placeholder title line. It has no setup or usage instructions yet.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server`. There is nothing to install or compile.

## Layout of `index.html`

- **`<head>`**: viewport meta set to lock zoom (`user-scalable=no`), theme color `#0b1d3a`, and an inline SVG favicon.
- **`<style>`** (around lines 10–361): all styles. Includes the `.reduced-motion` body class, which turns off most visual effects.
- **Markup** (lines 362–421): one `#container` holding the `#gameCanvas` canvas, HUD elements (`#hud`, `#hearts`, `#bombs`, `#weaponLvl`), boss bar (`#bossBarWrap`), combo and stage popups, touch controls (`#touchControls`, with `#moveZone`, `#joystickBase`, `#fireBtn`, `#bombBtn`), and the start/pause overlay (`#overlay`, `#overlayCard`).
- **Script** (lines 422–1668): a single IIFE. Everything lives inside it and nothing is exported to `window`.

Main sections of the script, in order:

| Area | Key identifiers |
| --- | --- |
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (localStorage keys prefixed `skyfighter.`) |
| Accessibility | `announce()` writes to the `#srLive` live region |
| Adaptive quality | `trackPerf()` and `quality` (0 full, 1 medium, 2 low) lower effects when frame time is high |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx.*`, `EXPLODE_SFX`. Synthesized with the Web Audio API; no audio files |
| Layout | `BASE_W = 480`, `BASE_H = 720`, `resize()` scales the canvas to the window |
| Input | Keyboard handlers, `joystickStart/Move/End()`, `canvasPointFromEvent()`, touch and mouse fire handlers |
| Game state | `resetGame()`, `update(dt)`, `draw()`, `loop(now)` (requestAnimationFrame; `dt` clamped to 50 ms) |
| Entities | `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()`, `fireWeapon()` |
| Power-ups | `applyPowerUp()` handles `life`, `bomb`, and `weapon` |
| Bosses | `BOSS_PATTERNS` (bullet patterns such as radial, spiral, cross, homing), `updateBoss()`, `killBoss()` |
| Flow | `loseLife()`, `endGame()`, `togglePause()` |

## Controls

- **Move:** Arrow keys or WASD. On touch, use the joystick in the left zone.
- **Fire:** Hold Space, or hold the fire button (mouse or touch).
- **Bomb:** `X` or either Shift key. Touch has a bomb button.
- **Pause:** `P` or `Escape`.

Space is `preventDefault`-ed on keydown so the page does not scroll.

## Game rules and tuning

- Each difficulty (`easy`, `normal`, `hard`) sets spawn rate, enemy shot chance, starting lives, starting bombs, and boss HP multiplier. Tune these in `DIFFICULTIES`.
- A stage ends when `KILLS_PER_STAGE` (18) kills are reached. A boss then spawns (`spawnBoss()`); defeating it advances to the next stage. Themes (`THEMES`) and bosses (`BOSSES`) cycle with `(stage - 1) % length`. Boss HP is `70 + stage * 35`, scaled by the difficulty and boss multipliers.
- Enemies: `grunt` (1 HP), `interceptor` (1 HP, weaves), `tank` (4 HP, appears from stage 2). See `ENEMY_TYPES`.
- Weapons: `vulcan` (spread shot) and `laser` (piercing lanes). Power-up `weapon` raises the level up to 3, then toggles the weapon type and resets the level to 1.
- Score combos: `addScore()` raises the multiplier by one for every 5 chained kills, capped at x10. `breakChain()` resets it. Multiplier is shown in the HUD.
- The best score is saved in localStorage under `skyfighter.bestScore`.

## Conventions

- Keep the game in a single file unless there is a clear reason to split it. The no-build, open-and-play setup is intentional.
- Follow the existing style inside the IIFE: `const`/`let`, arrow functions for callbacks, section comments written as `// ---------- Name ----------`, and short helper functions.
- Keep game-tuning numbers in the data objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Any new visual effect should respect `reducedMotion` and the adaptive `quality` level.
- Any new user-facing status message should go through `announce()` so screen readers get it.
- Keep touch and keyboard controls in parity when adding input.

## Testing

There is no test suite, linter, or CI configuration in the repo. Verify changes by loading `index.html` in a browser and playing through the affected path: start, fire, bombs, pause, stage transitions, boss fights, and game over. Check both desktop keyboard controls and touch layout when changing input or HUD code.

## Known gaps

- `README.md` contains only a title and a stray timestamp line. Update it with a real description and run instructions when convenient.
