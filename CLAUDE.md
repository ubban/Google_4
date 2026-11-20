# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html` (about 1,670 lines): markup at the top, CSS in a `<style>` block, and the game in a single inline `<script>`. `README.md` is a placeholder.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Do not add tooling unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. For touch-layout changes, check a phone-sized viewport as well as desktop.

## Code layout (`index.html`)

- **DOM**: `#container` holds `#gameCanvas`, the HUD (`#hud`, `#bossBarWrap`, `#comboPop`, `#stageBanner`), touch controls (`#touchControls`), and the start/pause/game-over card (`#overlay`).
- **Constants** (inside the script, near the top):
  - `DIFFICULTIES`: `easy` / `normal` / `hard`, controlling spawn rates, enemy speed, shot chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes, one per stage, cycled as the stage advances.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, appears from stage 2).
  - `BOSSES`: six bosses, each with its own colours and HP multiplier.
  - `BASE_W` × `BASE_H` (480 × 720): logical resolution. `resize()` scales the canvas to fit the window (capped at 1.6×).
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should tolerate storage being unavailable.
- **Audio**: Web Audio API, created lazily in `ensureAudio()` on the first user gesture. Sound effects are built from `beep()` and `noiseBurst()` and grouped in the `sfx` object.
- **Game loop**: `loop(now)` runs on `requestAnimationFrame` and calls `update(dt)` then `draw()`. `updateBoss(dt)` handles boss logic. `endGame()` and `togglePause()` manage state transitions. `trackPerf()` lowers the `quality` level (full / medium / low) when frame times are poor.
- **Accessibility**: `prefers-reduced-motion` is respected and can be toggled in-game. Screen-reader announcements go through `announce()` into the `#srLive` region.

## Controls

- Move: arrow keys or WASD. Mouse or touch also moves the ship.
- Fire: Space (hold for auto-fire). Touch uses the fire button.
- Bomb: X or Shift. Touch uses the bomb button.
- Pause: P or Escape.

## Conventions

- Keep the game as a single file unless the user asks for a split. Keep new code in the existing style: `const` by default, DOM references grouped at the top of the script, constants in SCREAMING_CASE.
- Gameplay numbers belong in the constant tables above, not hard-coded in the update logic.
- Any new HUD text or state change should also call `announce()` so screen-reader users get it.
- Keep the layout working at phone width (the page uses `user-scalable=no` and `viewport-fit=cover`).
- Don't introduce external network requests or CDN dependencies without asking.

## Git

- Development branch for this work: `claude/gifted-tesla-0zv9g9`. Don't push to other branches without explicit permission.
- Don't open pull requests unless the user asks.
