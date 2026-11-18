# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one ~1,670-line file. |
| `README.md` | Minimal placeholder (repo title plus a stray timestamp line). |
| `CLAUDE.md` | This file. |

There is no build step, package manager, test suite, or linter configured. Nothing needs installing.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000`).
- Keep the game working from `file://` as well as over HTTP. It currently loads no external resources (the favicon is an inline `data:` URI), so do not add modules, fetches, or external assets that would break that.

## Architecture of `index.html`

The file is organized top to bottom in this order:

1. **`<head>`**: meta tags (viewport locked to device width, `theme-color`), and the `<title>` "Sky Fighter".
2. **`<style>`**: all CSS. HUD, overlay, touch controls, and boss bar are styled here. Mobile layout is handled with `viewport-fit=cover` and touch-specific controls.
3. **`<body>`**: a single `<canvas id="gameCanvas">` plus DOM overlays for the HUD (score, best, multiplier, weapon, hearts, bombs), the start/game-over overlay, the boss bar, touch controls, and an accessibility live region (`srLive`).
4. **`<script>`** (one IIFE-style block, starting around line 422):
   - **DOM references**: `const xxxEl = document.getElementById(...)` collected at the top.
   - **Persistence keys**: `STORAGE_*` constants for `localStorage` (best score, mute, difficulty, reduced motion). Keep the `skyfighter.` prefix.
   - **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses, one per stage cycle).
   - **Audio**: a lazily created `AudioContext` (`ensureAudio`) with helpers `beep`, `noiseBurst`, and a `sfx` map. Audio is created on first user gesture.
   - **HUD and feedback**: `renderHearts`, `renderBombs`, combo pop, stage banner, screen shake, and flash effects.
   - **Input**: keyboard (`keys[e.code]`), mouse, touch joystick (`joystickStart/Move/End`), and fire/bomb buttons.
   - **Entities and logic**: `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `fireWeapon`, `update(dt)`, boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `updateBoss`, and `endGame`.
   - **Rendering**: `draw()` plus helpers `drawPlane`, `drawBossShip`, and `drawClouds`.
   - **Main loop**: `loop(now)` drives `update` and `draw` through `requestAnimationFrame`. A `trackPerf` sampler lowers a `quality` level (0 full, 1 medium, 2 low) when frame times are poor.

### Conventions used in the game code

- Logical resolution is `BASE_W = 480` by `BASE_H = 720`. `resize()` scales the canvas to fit the window, capped at 1.6x. Game coordinates should always be in logical pixels, never CSS pixels.
- Time-based movement uses `dt`. Keep movement frame-rate independent.
- Respect `reducedMotion`. Anything that adds screen shake, flashing, or flash overlays should check that setting.
- Accessibility: announce important state changes through `announce()` / `srLive`, and keep controls keyboard-, mouse-, and touch-reachable.
- Balance changes belong in the tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`) rather than being hard-coded into logic.

### Controls (for reference when testing)

- Move: arrow keys or WASD (via `keys`), pointer over the canvas, or the on-screen joystick on touch devices.
- Fire: hold Space, hold mouse button on the canvas, or hold the fire button. Weapon is `VULCAN` or `LASER`.
- Smart bomb: `X` or either Shift key (or the bomb button).
- Pause: `P` or `Escape`.
- Mute and motion toggles are in the HUD.

## Working in this repo

- **Make minimal, targeted edits.** The whole game lives in one file, so keep diffs focused and avoid reformatting unrelated sections.
- **Preserve the single-file, no-dependency design** unless the user explicitly asks for a change. Do not introduce npm, bundlers, or CDN scripts without asking.
- **Verify in a browser.** There are no automated tests, so for gameplay or rendering changes, load `index.html` in a browser (a headless Chromium with Playwright works) and check the affected behavior, including the start screen, a stage transition, and game over.
- **Check both input paths** when touching controls: keyboard and touch handlers are separate code paths.
- **Check mobile layout** at phone widths when changing CSS, since the HUD and touch controls are positioned absolutely.
- Keep `localStorage` access wrapped defensively, since it can throw in restricted browser contexts.

## Git

- Development branch for this session: `claude/gifted-tesla-cxquwv`. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Commit messages should be short and descriptive in the imperative mood (the existing history uses messages like "Add Sky Fighter: ...").
- Never commit secrets or local settings files.

## Keeping this file current

Update this CLAUDE.md when:

- files are added, removed, or renamed, or the project gains a build step, package manager, or test runner (update the commands and the file table);
- the game's structure changes significantly (new top-level sections, new entity types, new boss patterns, new storage keys);
- conventions or controls change.

Keep it accurate rather than exhaustive. Line numbers drift quickly, so refer to function and constant names instead.
