# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up played in the browser. It is a single self-contained file:

- `index.html` — all markup, CSS (`<style>`, ~lines 10–362), canvas element, and game logic (one `<script>` IIFE, ~lines 422–1669).
- `README.md` — a one-line title only.

There is no build step, package manager, bundler, test suite, or linter. Do not introduce any of these unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify changes by playing the game in a browser. Check at least: starting a run, shooting, a bomb, a stage transition, a boss fight, game over, and the best-score update.
- Also check the mobile layout (touch joystick, FIRE and BOMB buttons) when touching HUD, overlay, or touch code.

## Code structure (inside `index.html`)

The script is organized by `// ---------- Section ----------` comments. Keep new code in the matching section.

- **Element refs and constants** — DOM lookups at the top of the IIFE; `STORAGE_*` keys (`skyfighter.*`); `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Adaptive performance** — `trackPerf` moves a `quality` level (0–2) based on average frame time; drawing code should respect it.
- **Audio** — a Web Audio `sfx` object built on `beep`, `noiseBurst`, and `ensureAudio`. Audio starts only after a user gesture, so call `ensureAudio()` from input handlers.
- **HUD and feedback** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
- **Canvas sizing** — fixed logical size `BASE_W × BASE_H` (480 × 720), scaled by `resize()`. All game coordinates use this logical space.
- **Input** — keyboard (`keys[e.code]`), mouse/pointer (`canvasPointFromEvent`), and touch (joystick, fire and bomb buttons).
- **Game state and update** — `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore` / `breakChain` (kill-chain multiplier), `loseLife`, `endGame`, `togglePause`.
- **Boss patterns** — `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`); `updateBoss` and `killBoss` drive them.
- **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
- **Main loop** — `loop(now)` clamps `dt` to 0.05 s and calls `update` then `draw` only while running and not paused.

## Controls (keep the in-game hint in sync)

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire.
- X or Shift: bomb. P or Esc: pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

The on-screen text lives in `#controlsHint` (`index.html`, ~line 417). Update it whenever a control changes.

## Persistence

`localStorage` holds `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Reads and writes are unguarded, so keep the key names stable or add migration logic if you rename them.

## Conventions

- Plain HTML, CSS, and vanilla JavaScript (ES2015+ syntax already used: `const`/`let`, arrow functions, template literals). Do not add frameworks or external dependencies without asking.
- Keep everything inside the existing IIFE and `'use strict'` mode. Do not add globals.
- Match the existing style: 2-space indent, single quotes in JS, `const` by default, short section banners.
- Respect `reducedMotion`: anything that shakes, flashes, or pulses should check the flag or the `reduced-motion` body class.
- Keep gameplay numbers in the data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Use the `Sky Fighter` naming in user-facing text.

## Git

- Develop on the branch assigned to the session; do not push to other branches without permission.
- Write clear commit messages that describe the gameplay or code change.
- Do not create pull requests unless explicitly asked.
