# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-file HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), and JavaScript (`<script>`, ~lines 422–1667). No external dependencies.
- `README.md` — minimal placeholder.

There is no build step, package manager, bundler, linter, or test suite. Nothing to install.

## Running the game

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`) and visit it.
- Test on desktop (keyboard) and a touch device or touch emulation (joystick + fire/bomb buttons). Both control schemes are first-class.

## Code layout inside `index.html`

The script is one IIFE (`'use strict'`). Keep new code inside it; do not add globals.

Major sections, in file order:

1. **DOM references** — `const xxxEl = document.getElementById(...)` block at the top.
2. **Persistence keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`). Read/write only through `localStorage`.
3. **Adaptive quality** — `trackPerf()` lowers/raises a `quality` level (0–2) based on frame-time averages.
4. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`. Balance changes belong here, not scattered in logic.
5. **Audio** — Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on first user gesture; keep it that way for autoplay policies.
6. **HUD/feedback** — hearts, bombs, combo pop, stage banner, screen shake, hit/bomb flashes.
7. **Input** — keyboard listeners on `window`, pointer mapping via `canvasPointFromEvent`, touch joystick and buttons.
8. **Game state & entities** — `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `spawnExplosion()`, scoring via `addScore()` / kill-chain `breakChain()`, `useBomb()`, `fireWeapon()`.
9. **Update** — `update(dt)` advances all entities, collisions (`rectsOverlap`), and stage progression. `applyPowerUp()` handles pickups.
10. **Boss patterns** — `BOSS_PATTERNS` array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, etc.), driven by `updateBoss()` / `killBoss()`.
11. **Lifecycle** — `loseLife()`, `endGame()`, `togglePause()`.
12. **Render** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Pure drawing; should not mutate game state.
13. **Main loop** — `loop(now)` clamps `dt` to 50 ms, runs `update` then `draw` when running and not paused. Started with `requestAnimationFrame(loop)`.

## Conventions

- **Fixed logical resolution:** `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit; game coordinates stay in the base space.
- **Time-based motion:** all movement and timers use `dt` (seconds). Don't tie speeds to frame counts.
- **Style:** 2-space indentation, single quotes, `const` by default, `let` only when reassigned, arrow functions for callbacks, descriptive `camelCase` names. Section headers use `// ---------- Name ----------`.
- **No frameworks or dependencies.** Keep it vanilla HTML/CSS/JS in the single file unless the user asks for a restructure.
- **Accessibility:** honor the reduced-motion setting (`body.reduced-motion` class, `reducedMotion` flag) when adding screen shake, flashes, or other motion effects. Announce important state changes through `announce()` to the `srLive` region.
- **Gameplay numbers** (spawn rates, HP, scores, bomb counts) should live in the tuning tables at the top of the script.

## Making changes

- Read the relevant section before editing; the file is large (~1,670 lines), so use targeted searches (e.g. `grep -n "function update" index.html`) rather than reading it all.
- Verify changes by loading the page in a browser and playing through at least: start, shooting, a bomb, a power-up, a stage transition, a boss fight, and game over. Check the console for errors.
- Confirm that pause (`togglePause`), mute, difficulty switch, and best-score persistence still work after edits to shared state.
- Check both desktop and narrow/mobile viewports after any HUD or layout change.

## Git

- Development branch: `claude/gifted-tesla-m43nrn`. Commit and push there; do not push to other branches without explicit permission.
- Keep commits focused. Write commit messages in the imperative mood with a short summary line.
- Do not create pull requests unless explicitly asked.
