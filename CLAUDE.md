# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html` (about 1,670 lines), with inline `<style>` and one inline `<script>` block wrapped in an IIFE. `README.md` is a placeholder with only the title.

There is no build step, no package manager, no bundler, and no test suite. Nothing needs installing.

## Running it

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Play-testing is the only automated-ish check available. Use the `run` skill or Playwright (pre-installed in the cloud environment) to drive the page if you need to confirm a change in a real browser.

## Verifying changes

Since there are no tests, do this before committing any change to the game logic:

1. Extract the inline script and syntax-check it:
   ```bash
   awk '/<script>/{f=1;next}/<\/script>/{f=0}f' index.html > "$SCRATCH/sf.js" && node --check "$SCRATCH/sf.js"
   ```
   Use the scratchpad directory for `$SCRATCH`, not the repo.
2. Load the page in a browser and start a game to confirm there are no console errors.

## Code layout (`index.html`)

- **CSS** (lines ~6–360): `:root { color-scheme: dark; }`, HUD pills (`#hud .stat`), boss bar, overlay card, touch controls, and a `prefers-reduced-motion` path.
- **DOM** (lines ~362–420): `#container` holds the `#gameCanvas`, the HUD (`#hud`), boss bar, combo/stage banners, `#topBar` (mute, reduced-motion, pause buttons), `#touchControls` (joystick + FIRE/BOMB), and `#overlay` (start/pause/game-over card with difficulty buttons `.diffBtn`).
- **Script** (lines ~424 onward), all inside one IIFE:
  - DOM references (`canvas`, `overlay`, `scoreEl`, …) are cached at the top.
  - `STORAGE_*` keys and `localStorage` reads/writes: best score, mute, difficulty, reduced motion.
  - Audio: `ensureAudio()` lazily creates the `AudioContext`; `beep`, `noiseBurst`, and the `sfx` map provide all sound effects. No audio files.
  - Config tables: `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `THEMES` (background palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage), `KILLS_PER_STAGE` (18).
  - Input: `keys` map fed by `keydown`/`keyup`; joystick and touch handlers; mouse handlers on the canvas. Keyboard: arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
  - Game state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score, lives, bombs, stage, chain/multiplier.
  - Logic: `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `fireWeapon()` (vulcan with level-based spread, laser), `update(dt)`, `updateBoss(dt)`, boss patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, …), `useBomb()`, `endGame()`, `togglePause()`.
  - Rendering: `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`, with a `quality` level (0/1/2) that `trackPerf` lowers when frame times are slow.
  - Loop: a single `requestAnimationFrame` loop. `dt` is clamped to 50 ms. Update and draw run only when `running && !paused`.

Logical canvas size is `BASE_W` × `BASE_H` = 480 × 720. `resize()` scales it to the window, capped at 1.6×.

## Conventions

- Keep the game a single self-contained HTML file unless the user asks for a split. Do not add a build step or external dependencies. The only external resource is the inline `data:` SVG favicon.
- Keep any new script inside the existing IIFE. Do not add globals.
- Tuning values (speeds, spawn rates, HP, scores) belong in the config tables near the top of the script, not inline in logic.
- Preserve accessibility hooks: `#srLive` for screen-reader announcements via `announce()`, and the reduced-motion toggle that gates screen shake, flashes, and pulses.
- Preserve `prefers-reduced-motion` detection and the `touch-action`/`user-select` settings that keep mobile play from scrolling the page.
- Use the existing `localStorage` key prefix `skyfighter.` for any new persisted setting, and wrap storage access in try/catch if you add new code that touches it.

## Git

- Development branch: `claude/gifted-tesla-d0kbcd`. Commit and push there; do not push to `main` without explicit permission.
- Commit history is short (`Initial commit`, then the Sky Fighter import), so write clear, imperative commit messages describing the gameplay or code change.
