# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file, `index.html`, with inline CSS and one inline `<script>` block (an IIFE in strict mode). It draws to a `<canvas id="gameCanvas">` using the 2D context.

There is no build step, no bundler, no package manager, no framework, and no test suite. `README.md` is a placeholder.

## Repository layout

- `index.html` — the entire game: markup, styles, audio (Web Audio API), input handling, game loop, and rendering.
- `README.md` — placeholder text.
- `CLAUDE.md` — this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

Verify changes by playing the game in a browser, especially on touch/mobile layouts and with the keyboard.

## Code structure inside `index.html`

The script is organized roughly in this order:

1. **DOM references** — `document.getElementById(...)` lookups for the HUD, overlay, touch controls, and buttons.
2. **Storage keys** — `skyfighter.*` keys in `localStorage` for best score, mute, difficulty, and reduced motion.
3. **Helpers** — `announce` (screen-reader live region), `trackPerf`, audio helpers (`ensureAudio`, `beep`, `noiseBurst`, `panFor`), HUD renderers (`renderHearts`, `renderBombs`), and visual effects (`flashHit`, `screenShake`, `showComboPop`, `showStageBanner`).
4. **Config tables**
   - `DIFFICULTIES` — `easy`, `normal`, `hard`: spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` — kills needed to advance a stage.
   - `THEMES` — sky gradient, cloud, and sun colors per stage.
   - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank`, with size, HP, speed, score, and `minStage`.
   - `BOSSES` — six bosses (VANGUARD through TITAN) with colors and `hpMul`.
   - `BOSS_PATTERNS` — per-boss attack patterns built from `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`.
5. **Input** — keyboard listeners on `window`, pointer/touch joystick and button handlers, and `canvasPointFromEvent`.
6. **Game state and logic** — `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
7. **Rendering** — `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
8. **Main loop** — `loop(now)` drives `update` and `draw` each frame.

Game state lives in module-level `let` variables inside the IIFE (for example `player`, `bestScore`, `muted`, `difficulty`, `reducedMotion`). Nothing is exported.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a build system, bundler, or external dependencies without an explicit request.
- Preserve the single-file, no-network design. Avoid loading external scripts, fonts, or assets.
- Keep the IIFE and `'use strict'`. Do not add globals; attach new state to the existing closure.
- Follow the existing style: two-space indentation, single quotes in JavaScript, `const` by default, arrow functions for callbacks.
- Tuning values (speeds, spawn rates, HP, scores) belong in the config tables near the top of the script, not scattered through logic.
- Game time is driven by `dt` in `update(dt)`. Keep new movement and timers frame-rate independent.
- Respect the reduced-motion and mute settings when adding effects or sounds. Screen-reader announcements go through `announce`.
- Make the layout work on phones: the page uses `touch-action: none`, safe-area insets, and a touch joystick. Test changes at narrow widths.
- Persisted values use `localStorage` with `try`-less direct calls today; if you add new storage, keep the same `skyfighter.` key prefix.

## Git workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-ilosqk`). Do not push to other branches without permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.

## Verification checklist before committing

- The page loads without console errors in a desktop browser.
- Keyboard controls, pause, mute, and difficulty selection still work.
- Touch controls still work on a mobile-sized viewport.
- Game-over and best-score persistence still behave correctly.
- No new external network requests are introduced.
