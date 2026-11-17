# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only substantive content is **Sky Fighter**, a single-page HTML5 canvas vertical shoot-'em-up (Raiden-style) in `index.html`.

| File | Purpose |
| --- | --- |
| `index.html` | Entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines) |
| `README.md` | Minimal project readme |

There is no build step, package manager, test suite, or linter configured. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Because there is no test suite, verify changes by playing the game in a browser: start a run, check weapons/bombs/boss behavior, pause/resume, and check the mobile layout at phone width.

## Architecture of `index.html`

The file is structured top to bottom as:

1. **`<style>`** (lines 10–359): CSS for the HUD (pill-style, heart lives), glassmorphic overlay card, touch controls, and the reduced-motion class (`body.reduced-motion`).
2. **Markup** (around lines 360–420): `<canvas id="gameCanvas">`, HUD elements, overlay, touch controls (`moveZone`, `fireBtn`, `bombBtn`), and a hidden `srLive` region for screen-reader announcements.
3. **`<script>`** (starts ~line 422), wrapped in an IIFE with `'use strict'`:
   - **DOM refs** at the top of the IIFE. Add new element lookups here.
   - **Persistence keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. These live in `localStorage`. Keep key names stable, since changing them resets players' saved state.
   - **Tuning constants**: `DIFFICULTIES` (spawn rates, enemy speed/shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt / interceptor / tank), and `BOSSES` (six bosses with colors and `hpMul`).
   - **Audio**: a lightweight Web Audio layer (`ensureAudio`, `beep`, `noiseBurst`, `sfx.*`). Audio is created lazily on the first user gesture (`ensureAudio()`), so keep it that way for autoplay policies.
   - **Adaptive quality**: `trackPerf` samples frame times and steps a `quality` level (0–2) up or down to shed effects on slow devices.
   - **Input**: keyboard (`keys[e.code]`), a virtual joystick for touch, mouse steering/firing on the canvas, and bomb/fire buttons. Keyboard: arrows/WASD move, Space fires, X or Shift bombs, P/Esc pauses.
   - **Game logic**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `useBomb`, `fireWeapon`, `applyPowerUp`, `breakChain` / `addScore` (kill-chain score multiplier).
   - **Weapons and bosses**: `fireWeapon` handles the Vulcan/Laser systems at three power levels. Boss attacks are defined as `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring), driven by `updateBoss`.
   - **Rendering**: `drawPlane`, `drawBossShip`, and the parallax cloud/sun layers, all drawn to the canvas each frame.
   - **Main loop**: `loop` calls `requestAnimationFrame`; `update` and draw run only when `running && !paused`.

Game state is held in module-scope `let` variables inside the IIFE (e.g. `score`, `lives`, `running`, `paused`, `gameOver`). There are no modules, classes, or frameworks.

## Conventions

- **Single-file app.** Keep the game in `index.html` unless there's a clear reason to split it. If you split it, update every reference and keep the file deployable as static files.
- **No external dependencies.** The project uses no CDNs or npm packages. Keep it that way unless the user asks otherwise. The favicon is an inline SVG data URI.
- **Vanilla JS, ES2015+ style** with `const`/`let`, arrow functions, and template literals, matching the existing code.
- **Keep the IIFE and `'use strict'`.** Do not leak globals.
- **Naming:** `UPPER_SNAKE_CASE` for constants and tables, `camelCase` for functions and variables, `STORAGE_*` for `localStorage` keys, and `skyfighter.*` for their values.
- **Time-based updates:** movement and timers use the `dt` passed to `update(dt)`, so keep gameplay frame-rate independent.
- **Accessibility:** respect `prefers-reduced-motion` (via `reducedMotion` and the `reduced-motion` body class), and announce important events through `announce()` to the live region.
- **Mobile:** the layout must work at phone width without horizontal scroll. Touch controls are required alongside keyboard and mouse controls.
- **Comments:** section headers use `// ---------- Name ----------`. Keep comments short and explain *why*, not *what*.

## Making changes

- When changing tuning values, edit the constants tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in logic.
- Changing a `localStorage` key or the shape of stored data breaks existing players' saved best score or settings. Migrate or keep the old key if you must change it.
- Test in a real browser after any change to rendering, input, or the loop. Check pause/resume and game over, and confirm no console errors.
- Commit messages should be concise and describe the gameplay or code change.

## Git

- Develop on the branch assigned for the session. Don't push to other branches without permission.
- Don't open pull requests unless explicitly asked.
