# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`). ~1,670 lines.
- `README.md` — minimal project readme.

There is no build step, no package manager, no bundler, and no test suite. Do not introduce one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```sh
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Verifying changes

- There are no automated tests or linters configured. Verify changes by loading the game in a browser and playing through the affected path (start, fire, bomb, pause, boss, game over).
- For UI or gameplay changes, a headless Playwright run is a reasonable check; Chromium is usually preinstalled in this environment.
- Keep the file syntactically valid JavaScript: the game runs inside a single IIFE (`(function () { 'use strict'; ... })();`) and any syntax error stops the whole game.

## Code structure (`index.html`)

The file is organized in this order:

1. **`<head>`** — viewport meta (locks zoom, uses `viewport-fit=cover` for notched phones), theme color, title.
2. **`<style>`** — all styling, including the HUD, overlay card, touch controls, and screen-reader-only helper (`.sr-only`).
3. **Markup** — `#container` holds the `<canvas id="gameCanvas">`, screen-flash/vignette layers, HUD (`#hud`), boss bar, stage/combo banners, top-right icon buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls, and the start overlay (`#overlay`).
4. **Script** — one IIFE containing:
   - DOM lookups (top of the IIFE). Add new element references here.
   - **Persistence keys** — `STORAGE_*` constants prefixed `skyfighter.` (best score, mute, difficulty, reduced motion). Changing a key name resets that user's saved value.
   - **Adaptive performance** — `trackPerf` steps a `quality` level (0–2) based on rolling frame time.
   - **Config tables** — `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here.
   - **Audio** — a lazily created `AudioContext` (`ensureAudio`), with `beep` / `noiseBurst` helpers and a `sfx` object of named sounds. Audio only starts after a user gesture.
   - **Input** — keyboard (`keys[e.code]`), touch joystick + FIRE/BOMB buttons, and mouse steering/firing on the canvas.
   - **Game state and logic** — `resetGame`, `update(dt)`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, `applyPowerUp`), bombs (`useBomb`), scoring/combos (`addScore`, `breakChain`), boss attack patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`), life loss and game over (`loseLife`, `endGame`), and pause (`togglePause`).
   - **Rendering** — `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`.
   - **Main loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` while running and not paused. Starts via `requestAnimationFrame(loop)`.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. New assets should be inline (SVG/CSS/code-drawn), not external files, to match the current design.
- **Canvas drawing is procedural.** Sprites are drawn with canvas path calls, not image files. Follow the existing helper style when adding entities.
- **Use the config tables** for tunable values (enemy stats, difficulty multipliers, boss HP multipliers, themes) rather than hard-coding numbers in logic.
- **Respect reduced motion.** Screen shake, flashes, and pulses are gated by `reducedMotion`. New visual effects that move the screen or flash should check it too.
- **Accessibility.** Keep ARIA labels and `title` attributes on buttons, and announce important state changes through `announce()` (which writes to `#srLive`).
- **Mobile and desktop both matter.** Any new control must work with both touch and keyboard/mouse. The canvas is scaled from a fixed `BASE_W × BASE_H` (480 × 720) via `resize()`; keep game coordinates in that space.
- **Keep the game loop frame-rate independent.** Movement and timers use `dt` (seconds). Don't add per-frame fixed increments without scaling by `dt`.
- **Browser storage is per-origin and optional.** Wrap new `localStorage` access if it could be unavailable (private mode, blocked storage).

## Controls (for reference when testing)

| Action | Desktop | Mobile |
| --- | --- | --- |
| Move | Arrow keys / WASD, or mouse over canvas | Drag left zone (joystick) |
| Fire | Space (hold), or hold mouse button | Tap/hold FIRE |
| Bomb | X or Shift | Tap BOMB |
| Pause | P or Esc (or ⏸ button) | ⏸ button |

## Git and branches

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-8p6dj1`). Do not push to other branches without explicit permission.
- Commit only when asked or when a task explicitly calls for it, and keep commit messages focused on the change.
- Do not open a pull request unless the user explicitly asks for one.
