# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one self-contained file, `index.html` (about 1,670 lines), with CSS in a `<style>` block and JavaScript in one `<script>` IIFE. There are no external dependencies, no bundler, and no package manager.

Other files:
- `README.md`: a placeholder heading only.

## Running and testing

- There is no build step, lint config, or test suite. Do not invent commands for them.
- To run the game, open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Verify changes by playing the game in a browser. Check at least: mouse, keyboard, and touch controls; pause and resume; a game over and a restart; and the mute, reduced-motion, and difficulty toggles.
- For a headless check, Playwright with the pre-installed Chromium works. Load the page, click `#startBtn`, and inspect the DOM or console.

## Code layout (`index.html`)

- **Markup and CSS (lines ~1–420):** the canvas (`#gameCanvas`), HUD elements (score, hearts, bombs, weapon, boss bar), overlay, touch controls, and the controls hint text. Styles are inline in `<style>`.
- **Script (lines ~422–1668):** one IIFE with `'use strict'`. Sections, roughly in order:
  - DOM references and `STORAGE_*` keys.
  - Adaptive performance (`trackPerf`) that lowers `quality` when frame time is high.
  - Tuning tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`.
  - Audio: a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a named `sfx` set.
  - HUD helpers: hearts and bombs rendering, flashes, combo pops, stage banners, and screen shake.
  - Input: keyboard (`keys`), a virtual joystick and buttons for touch, and mouse steering on the canvas.
  - Game state and `resetGame`, the update loop (`update`), drawing (`draw`, `drawPlane`, `drawBossShip`), spawning, weapons and bullet patterns, boss logic (`updateBoss`, `killBoss`), `loseLife`, `endGame`, and pause.
  - `loop(now)`: a `requestAnimationFrame` loop that clamps `dt` to 0.05 s, then calls `update` and `draw` while running and not paused.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Adding a build step or external dependencies is a significant change and should be discussed first.
- Follow the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, and short helper functions with descriptive camelCase names.
- Put tunable numbers in the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Persistent settings use `localStorage` with the `skyfighter.*` key prefix. Read and write them the same way the existing code does.
- Accessibility matters here. Announce important state changes through `announce()` (the `#srLive` live region), and respect `reducedMotion`, which is set from the user's preference or the in-game toggle.
- Keep the mobile layout working. The game uses a fixed 480×720 logical canvas (`BASE_W`, `BASE_H`) that is scaled in `resize()`.

## Gameplay reference

- **Controls:** mouse steers and clicks to fire; Arrows or WASD move; Space fires; X or Shift drops a bomb; P or Escape pauses. On touch, drag the left zone to fly, then tap FIRE or BOMB.
- **Difficulty:** `easy`, `normal` (default), and `hard`. They change spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Progression:** clearing `KILLS_PER_STAGE` (18) kills spawns a boss. Defeating the boss advances to the next stage and cycles through `THEMES` and `BOSSES`.
- **Enemies:** `grunt`, `interceptor` (fast, weaves), and `tank` (4 HP, appears from stage 2).
- **Power-ups:** life, bomb, and weapon. Weapon power-ups raise the level up to 3, then switch between `vulcan` and `laser`.
- **Scoring:** points come from kills with a chain multiplier. Taking a hit breaks the chain. The best score persists in `localStorage`.

## Working agreements

- Before editing, read the surrounding code. Many sections share state variables such as `enemies`, `bullets`, `powerUps`, `boss`, and `running`.
- When you change game state, check every path that touches it: `resetGame`, `endGame`, pause and resume, and the touch and keyboard inputs.
- Do not commit generated or temporary files. The repository currently has no `.gitignore`.
- Commit with clear messages, and push to the branch you were given.
