# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html`. `README.md` is a placeholder.

There is no build step, no package manager, no dependencies, and no test suite.

## Running it

- Open `index.html` directly in a browser, or serve the directory with any static file server.
- Verify changes by playing them in a browser. Nothing else in the repo checks behavior.

## File layout (`index.html`, ~1,670 lines)

| Lines (approx.) | Contents |
| --- | --- |
| 1–361 | `<style>`: all CSS, including the HUD, overlay, touch controls, and responsive layout |
| 362–421 | Markup: canvas, HUD, boss bar, overlay card with difficulty buttons, touch controls |
| 422–1668 | One `<script>` containing a single IIFE (`(function () { 'use strict'; ... })()`) |

Inside the IIFE, the script is organized by section comments, in this order:

1. **DOM refs and storage keys**: `const` lookups for every element, plus `STORAGE_*` keys.
2. **Adaptive performance**: `trackPerf` moves a `quality` level (0–2) up or down based on a 45-frame rolling average.
3. **Tuning data**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Most balance changes belong here.
4. **Settings state**: best score, mute, difficulty, and reduced motion, all persisted through `localStorage`.
5. **Audio**: WebAudio helpers (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on the first user gesture.
6. **UI helpers**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce`.
7. **Layout and input**: `resize` (fixed 480×720 logical canvas, `BASE_W`/`BASE_H`), keyboard, mouse, touch joystick, and fire/bomb buttons.
8. **Game state and simulation**: `resetGame`, `update(dt)`, spawning, collisions, power-ups, scoring and chains, bombs, weapons, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
9. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
10. **Main loop**: `loop(now)` runs on `requestAnimationFrame`, caps `dt` at 0.05 s, then calls `update(dt)` followed by `draw()`. Start-button wiring is at the bottom.

## Game mechanics worth knowing

- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rates, enemy speed and fire chance, starting lives and bombs, and a boss HP multiplier.
- **Stages**: after `KILLS_PER_STAGE` (18) kills, a boss spawns. Defeating it advances the stage. The theme comes from `THEMES[(stage - 1) % THEMES.length]`.
- **Enemies**: `grunt`, `interceptor` (weaves), and `tank` (4 HP, unlocked at stage 2). `spawnEnemy` picks from the types whose `minStage` is reached.
- **Bosses**: one per stage, chosen by `BOSSES[(stage - 1) % BOSSES.length]`. Each boss's attack pattern comes from `BOSS_PATTERNS`.
- **Power-ups** (`life`, `bomb`, `weapon`) come from `maybeSpawnPowerUp`.
- **Score**: base values per enemy, multiplied by a chain multiplier that `breakChain` resets.

## Input

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: move over the canvas to steer; hold the mouse button to fire.
- Touch: drag the left zone for the virtual joystick; tap FIRE and BOMB.

Keyboard state is stored in `keys[e.code]`. Touch and mouse fire use the `firing` flag.

## Persistence and accessibility

- `localStorage` keys use the `skyfighter.*` prefix: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes are direct calls and are not wrapped in try/catch. If you add storage access that can run in private or restricted contexts, wrap it.
- Screen-reader announcements go through `announce()`, which writes to the `#srLive` aria-live region. Use it for new state changes that matter to the player.
- Reduced motion is user-toggleable and also respects `prefers-reduced-motion`. Gate new screen shake, flashes, and particle effects on it.

## Conventions

- Vanilla HTML, CSS, and JavaScript. Do not add a framework, bundler, or npm dependency without asking first.
- Keep new JavaScript inside the existing IIFE. Do not add globals.
- Match the existing style: `const`/`let`, 2-space indentation, single quotes, arrow functions for callbacks, and short section comments in the `// ---------- Name ----------` form.
- Keep the game responsive. Test any layout or input change at phone width and with touch controls, not just with the keyboard on desktop.
- Keep the 480×720 logical resolution unless a change requires otherwise. `resize()` handles scaling.
- Balance tweaks should change the constants in the tuning tables, not the logic.

## Git

- Develop on the branch you were assigned. Do not push to other branches without permission.
- Commit with clear messages that describe the change.
- Do not open a pull request unless asked.
