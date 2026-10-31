# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs entirely in the browser.

- `index.html` — the whole game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines).
- `README.md` — minimal project title only.

There is no build system, package manager, bundler, test suite, or server-side code. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code layout (inside `index.html`)

- **`<head>`** — viewport meta (disables user zoom, `viewport-fit=cover` for notched phones), theme color, and page title.
- **`<style>`** — HUD (pill-style hearts/bombs/score), glassmorphic overlay card, touch controls, and animations.
- **Body markup** — `<canvas id="gameCanvas">`, HUD elements, overlay (title/start/game-over), touch joystick and fire/bomb buttons, pause and mute/motion toggles, and an ARIA live region (`#srLive`).
- **`<script>`** — a single IIFE (`'use strict'`) containing all game logic. Key sections, in file order:
  - Config tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`.
  - Persistence: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
  - Audio: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`, `panFor`).
  - Input: keyboard handlers, joystick, and pointer mapping (`canvasPointFromEvent`).
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, and the main `draw()`.
  - Simulation: `update(dt)`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `loseLife`, `endGame`.
  - Loop: `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 0.05 s, and calls `update` then `draw` only when running and not paused.

## Game design reference

- **Controls:** move with `WASD` or arrow keys (or touch joystick); fire with `Space`; bomb with `X` or `Shift`; pause with `P` or `Escape`. Touch devices get on-screen fire/bomb buttons and a joystick.
- **Logical resolution:** `BASE_W = 480`, `BASE_H = 720`, scaled to fit the window in `resize()`.
- **Weapons:** Vulcan/Laser system with 3 power levels, switched through power-ups.
- **Bombs:** limited smart bombs that clear the screen and damage bosses.
- **Stages:** waves of enemies; after `KILLS_PER_STAGE` kills a boss warning fires and a boss from `BOSSES` fights with its own attack patterns (`BOSS_PATTERNS`) and a health bar.
- **Scoring:** kill-chain multiplier (`addScore`, `breakChain`).
- **Difficulty:** `easy` / `normal` / `hard` via `DIFFICULTIES`, persisted to `localStorage`.
- **Performance:** `trackPerf` samples frame time over 45 frames and adjusts a `quality` level up or down to keep frame rate up.
- **Accessibility:** respects `prefers-reduced-motion` on first load (user can override with the motion toggle). When reduced, screen shake is damped and shortened, and hit/bomb flashes are suppressed. Announcements go through `#srLive`.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Preserve the single-file, no-dependency design.
- Use plain ES2017+ JavaScript and the Canvas 2D API. Do not add frameworks, CDN scripts, or npm dependencies without asking.
- Keep the IIFE wrapper and `'use strict'`. Do not introduce new globals.
- Keep configuration in the existing `const` tables (`DIFFICULTIES`, `BOSSES`, `ENEMY_TYPES`, etc.) rather than hard-coding magic numbers in logic.
- Persist new player settings under the `skyfighter.` localStorage prefix, and wrap any new storage access so the game still works when storage is unavailable.
- Preserve mobile support: any new control needs a touch path and must respect the `viewport` and `touchControls` layout.
- Match the existing indentation (2 spaces) and quote style (single quotes in JS).

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a desktop browser and check that the title screen, start, pause (`P`), bomb (`X`), and game-over flow still work.
2. Check the difficulty buttons, mute toggle, and motion toggle, and reload the page to confirm settings persist.
3. Check a touch-sized viewport (for example, via DevTools device mode) for layout and joystick behavior.
4. Confirm there are no console errors on load or during play.

Headless browser checks are possible with the pre-installed Playwright/Chromium in the cloud environment, if a scripted check is needed.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-0gpync`); do not push to `main` directly.
- Write clear, descriptive commit messages that explain the why.
- Do not open a pull request unless explicitly asked.
