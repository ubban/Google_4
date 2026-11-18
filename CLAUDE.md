# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines): all markup, CSS (`<style>`), and game logic (one inline `<script>` wrapped in an IIFE). No build step, no dependencies, no bundler, no tests.
- `README.md`: a placeholder title only.

There is no `package.json`, no test suite, and no linter configured. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Architecture of `index.html`

**Markup / CSS (lines ~1–420):** A `#container` holds a `<canvas id="gameCanvas">` (logical size 480×720, `BASE_W`/`BASE_H`, scaled to fit by `resize()`), a HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), the boss bar, the top-bar icon buttons (mute, reduced-motion, pause), touch controls (`#touchControls`: joystick + FIRE/BOMB), and the start/game-over `#overlay`. An `aria-live` region (`#srLive`) carries screen-reader announcements via `announce()`.

**Script (lines ~422–1669), in order:**

1. **DOM refs and constants** — element handles, `localStorage` keys (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`).
2. **Tuning data** — `DIFFICULTIES` (easy/normal/hard: spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled per stage), `BOSS_PATTERNS` (bullet patterns).
3. **Adaptive quality** — `trackPerf()` samples frame time over 45 frames and steps `quality` 0–2 to reduce effects when frames are slow.
4. **Audio** — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). No audio files. `ensureAudio()` must run from a user gesture.
5. **Input** — keyboard (Arrow keys or WASD to move via `keys[e.code]`, Space to fire, X/Shift for bomb, P/Esc to pause), mouse (`mousemove`/`mousedown` on the canvas, plus hold-to-fire on the FIRE button), and touch joystick + buttons. Touch controls show only when `isTouchDevice`.
6. **Simulation** — `resetGame()`, `update(dt)` (movement, firing, collisions, spawning, scoring, combo chain), `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp` (life / bomb / weapon upgrade), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
7. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Screen shake, hit/bomb flashes, and combo/stage banners are DOM overlays, gated by `reducedMotion`.
8. **Loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` + `draw` while `running && !paused`. `startBtn` starts a run.

**Persistence:** best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` (direct calls, no try/catch). Reduced motion defaults to the OS `prefers-reduced-motion` setting until the user toggles it.

## Working in this repo

- **Keep it a single file** unless the user asks for a split. Preserve the IIFE and `'use strict'`.
- **Game balance** lives in the top-of-script constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer tuning those over hard-coding numbers in logic.
- **Don't add dependencies, a build step, or a framework** without an explicit request. The project is intentionally zero-dependency.
- **Accessibility matters here:** new on-screen feedback should respect `reducedMotion`, and important state changes should go through `announce()`.
- **Mobile:** the layout must keep working at phone widths and under touch input. Test touch paths when changing controls.
- **Verification:** there are no automated tests. For gameplay or rendering changes, load the page in a browser (the `run` skill or Playwright, with `--no-sandbox` if needed) and check the canvas, HUD, and console for errors. A syntax-only check is `node --check` on the extracted script if needed.

## Git

- Development branch for this session: `claude/gifted-tesla-zxxz7s`. Push there with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.
- Commit messages follow the existing short imperative style (e.g. "Add Sky Fighter: ...").
