# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser. The entire game is a single self-contained file, `index.html` (inline `<style>`, markup, and one inline `<script>`, about 1,670 lines). There is no build step, no package manager, no bundler, and no test suite.

Other files:
- `README.md`: a placeholder title only. Keep it minimal unless asked to expand it.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There is nothing to install or compile.

## Verifying changes

- No automated tests or linters are configured. Check changes by loading the page in a browser and playing, or with a headless browser such as Playwright (Chromium is preinstalled in the cloud environment).
- Before finishing a change, run the inline script through a syntax check. For example, extract the contents of the `<script>` block into a temp file and run `node --check` on it.
- Test both keyboard and touch paths when touching input code, and check the pause, game-over, and restart flows.

## Code structure (`index.html`)

- **Head and style (lines ~3–360)**: all CSS lives inline. Colors and layout are tuned for a 480×720 logical canvas (`BASE_W`, `BASE_H`), scaled to fit the viewport in `resize()`. The file has no CSS `@media` queries; responsive behavior is handled in JS.
- **Markup (lines ~361–421)**: a single `<canvas id="gameCanvas">`, the HUD (score, best, multiplier, weapon, hearts, bombs), overlays (title, pause, game over), boss health bar, touch controls (joystick, fire, bomb), and a screen-reader live region (`#srLive`).
- **Script (lines ~422–end)**: wrapped in one IIFE with `'use strict'`. Sections, in order:
  - DOM references and storage keys
  - Adaptive performance (`trackPerf`): lowers effect quality when frame time is high
  - Config tables: `DIFFICULTIES`, `THEMES` (one per stage, cycling), `ENEMY_TYPES`, `BOSSES`
  - Settings persisted to `localStorage`
  - Audio: a lazily created Web Audio context (`ensureAudio`), with `beep` and `noiseBurst` helpers. Audio must start from a user gesture.
  - Input: keyboard (`keys` map), virtual joystick, mouse and touch on the canvas, and fire and bomb buttons
  - Game state and `resetGame()`
  - Rendering: `drawPlane`, `drawBossShip`, `draw()`, and cloud and sun layers
  - Spawning and combat: `spawnEnemy`, `spawnBoss`, `fireWeapon`, `maybeSpawnPowerUp`, `applyPowerUp`, and bombs (`useBomb`)
  - Boss patterns: `BOSS_PATTERNS` (an array of bullet-pattern functions), `updateBoss`, `killBoss`
  - Flow: `loseLife`, `endGame`, `togglePause`, and the main `loop(now)` driven by `requestAnimationFrame`

## Game rules and tuning

- **Controls**: move with arrow keys or WASD. Fire with Space (hold). Bomb with X or Shift. Pause with P or Escape. On touch, use the virtual joystick, fire button, and bomb button.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. These set spawn timing, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Stages**: each stage lasts `KILLS_PER_STAGE` (18) kills, after which a boss is spawned (`spawnBoss`). Bosses come from `BOSSES`, and the background theme comes from `THEMES`.
- **Enemies**: `grunt`, `interceptor` (weaving), and `tank` (4 HP, appears from stage 2).
- **Scoring**: a kill-chain multiplier (`chain`, `multiplier`) builds on consecutive kills and resets via `breakChain()`.
- **Persistence**: best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under keys prefixed `skyfighter.`. Keep these keys stable, or migrate them if they change, so players keep their best score.
- **Reduced motion**: defaults from `prefers-reduced-motion`, and can be toggled in-game (the 🎆/🚫 button). It reduces screen shake and flash.

## Conventions

- Keep the game as a single file unless the user asks for a restructure. Do not add a build system or dependencies without asking.
- Match the existing style: 2-space indentation, `const` and `let` (no `var`), semicolons, single quotes in JS, and short section comments in the `// ---------- Name ----------` style.
- Put new tuning values in the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Keep the update/draw split: game logic goes in `update(dt)` and `updateBoss(dt)`, and rendering goes in `draw()` and its helpers. Use `dt` (seconds, clamped to 0.05) for time-based motion.
- Use `quality` to gate expensive effects (shadows, extra particles, glow) for lower-end devices.
- Accessibility: announce important state changes through `announce()`, which writes to the `#srLive` region.

## Git

- Development branch: `claude/gifted-tesla-e49mvu`. Commit and push there unless told otherwise.
- Commit messages describe the gameplay or UI change in plain language. The existing history uses an imperative summary line followed by a bulleted body.
