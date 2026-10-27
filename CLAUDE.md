# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` is the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`, wrapped in one IIFE).
- `README.md` is a placeholder and does not describe the project.
- There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Repository layout

```
index.html   # The whole game (~1670 lines)
README.md    # Placeholder
```

Inside `index.html`, the file is organized in this order:

1. `<head>`: meta tags, inline SVG favicon, and the `<style>` block (CSS custom properties, HUD, overlay, touch controls, reduced-motion rules).
2. `<body>`: `#container` holding the `<canvas id="gameCanvas">`, HUD (`#hud`), boss bar (`#bossBarWrap`), combo/stage popups, top bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
3. `<script>`: one IIFE with `'use strict'`, organized into commented sections:
   - `// ---------- Adaptive performance ----------` (frame-time based quality tiers)
   - Constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`
   - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers
   - `// ---------- Input ----------` (keyboard, mouse, touch joystick, fire/bomb buttons)
   - Game state and logic: `resetGame`, `update`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`
   - Rendering: `draw`, `drawPlane`, `drawBossShip`, `drawClouds`
   - `loop(now)`: fixed-structure `requestAnimationFrame` loop, `dt` clamped to 0.05s

## Architecture notes

- **Rendering**: a single `<canvas>` with a fixed logical size of `480 x 720` (`BASE_W`, `BASE_H`). `resize()` scales the canvas via CSS to fit the window, capped at 1.6x. All game coordinates use the logical size; do not use CSS pixels in game logic.
- **Loop**: `update(dt)` runs only when `running && !paused`, followed by `draw()`. `dt` is capped at 0.05s so tab switches do not teleport entities.
- **Adaptive quality**: `trackPerf` averages the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low). Drawing code reads `quality` to reduce effects. Keep new expensive effects behind this check.
- **State**: game state lives in module-level `let` variables inside the IIFE (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `kills`, `chain`, `multiplier`, and so on). `resetGame()` is the single place that initializes a new run; add any new per-run state there.
- **Difficulty**: `DIFFICULTIES` sets spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and a boss HP multiplier. `setDifficulty()` persists the choice.
- **Stages**: reaching `KILLS_PER_STAGE` kills triggers a boss fight (`spawnBoss`). Defeating the boss (`killBoss`) increments `stage`. Each stage picks a theme from `THEMES` using `(stage - 1) % THEMES.length`. Bosses come from `BOSSES` (cycled the same way), and their attacks are the `BOSS_PATTERNS` functions.
- **Weapons**: `player.weapon` is `'vulcan'` or `'laser'`, and `player.level` is 1 to 3. `fireWeapon()` branches on both.
- **Scoring**: `addScore()` multiplies by a kill-chain `multiplier`; `breakChain()` resets it.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should keep working when storage is unavailable or throws; wrap new storage calls in try/catch if you add them.
- **Accessibility**: `announce()` writes to the `#srLive` live region. Reduced-motion mode (toggled with `#motionBtn`, defaults to `prefers-reduced-motion`) disables screen shake, flash, and pulse effects via the `reduced-motion` body class.
- **Audio**: Web Audio API, created lazily by `ensureAudio()`, which must run from a user gesture. Muting is persisted via `STORAGE_MUTE`.

## Controls (for manual testing)

- Mouse: move to steer, click to fire.
- Keyboard: arrows or WASD to move, Space to fire, X or Shift for bomb, P or Escape to pause.
- Touch: drag the left zone to fly, FIRE and BOMB buttons on the right (shown only on touch devices).

## Working in this repo

- Keep the game as a single file unless the user asks to split it. Match the existing style: 2-space indentation, single quotes, `const` by default, short section comments with the `// ---------- Name ----------` form.
- Do not introduce external dependencies, CDNs, or a build tool without asking.
- When changing balance values, update the relevant constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than hard-coding numbers in the logic.
- Verification: there are no automated tests. Verify changes by loading `index.html` in a browser and playing through the affected path (start, shooting, bombs, a stage transition, a boss fight, pause/resume, game over). Check the browser console for errors.
- Run `git status` before committing and keep commits focused.
