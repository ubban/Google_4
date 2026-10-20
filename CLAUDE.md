# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 canvas.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`, wrapped in one IIFE with `'use strict'`).
- `README.md` — a placeholder title. It has no setup or usage instructions yet.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install step is needed.

## Code layout inside `index.html`

- **Markup (~lines 1–420):** `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`), boss bar, stage and combo banners, top-bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`, `#fireBtn`, `#bombBtn`, joystick elements), and the start overlay (`#overlay`, `#startBtn`, difficulty buttons `.diffBtn`).
- **Styles:** a single inline `<style>` block in the `<head>`. Reduced-motion overrides are scoped under `body.reduced-motion`, which the in-game toggle sets.
- **Script:** one IIFE. Within it, roughly in order:
  - DOM lookups (`const xEl = document.getElementById(...)`).
  - Constants: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - Audio: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` object, all built on the Web Audio API.
  - HUD and effects helpers: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
  - Input: keyboard (`keys` map), mouse, and touch joystick handling. Bomb and fire are wired to both touch and click.
  - Game state: `resetGame()`, `update(dt)` (per-frame logic, variable `dt`), `draw()` (rendering), `loop(now)` (requestAnimationFrame driver, `dt` clamped to 50 ms).
  - Weapons and enemies: `fireWeapon`, `applyPowerUp`, `spawnEnemy`, `spawnBoss`, `updateBoss`, and the boss bullet-pattern functions referenced from `BOSS_PATTERNS`.
  - Flow: `killBoss`, `loseLife`, `endGame`, `togglePause`.
  - Adaptive quality: `trackPerf(dt)` lowers the `quality` level (0 full, 1 medium, 2 low) when the average frame time is slow, and raises it again when the game runs fast.

## Conventions

- **Persistence:** user settings and the best score are stored in `localStorage` under the keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Keep new keys under the `skyfighter.` prefix. Reads and writes must tolerate storage being unavailable.
- **Coordinates:** the game logic runs in a fixed `BASE_W × BASE_H` (480 × 720) space. `resize()` scales the canvas to fit the window. Do not hard-code device pixels.
- **Accessibility:** announce important state changes through `announce()`, which writes to the `aria-live` region `#srLive`. Respect `prefers-reduced-motion` through the in-game toggle, which defaults to the system preference. Any new screen shake, flash, or pulse must check `reducedMotion`.
- **Audio:** call `ensureAudio()` from a user gesture before playing sounds. Browsers block audio before that.
- **Style:** match the existing code. It uses 2-space indentation, single quotes in JavaScript, `const` by default, arrow functions for callbacks, and short section comments such as `// ---------- Name ----------`. Keep additions inside the existing IIFE. Do not add globals.
- **Tuning:** balance numbers (spawn rates, enemy HP, boss HP multipliers, lives, bombs) live in the constant tables near the top of the script. Change them there rather than inline.

## Making changes

- Make small, focused edits to `index.html`. Because the whole game is one file, a diff can touch several unrelated areas, so review it carefully.
- After changing gameplay or rendering code, open the page in a browser and play for a short while. Check the start overlay, a stage transition, a boss fight, pause, and game over. No automated test covers these.
- When you change a `localStorage` key, a difficulty value, or a DOM id that the script looks up, update this file to match.

## Git

- Develop on the branch the session names (currently `claude/gifted-tesla-psfw58`). Do not push to other branches without permission.
- Do not open a pull request unless the user asks for one.
