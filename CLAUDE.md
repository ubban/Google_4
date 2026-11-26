# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser.

- `index.html` — the entire game: markup, CSS, and one inline `<script>` (game logic, rendering, audio, input). No external dependencies.
- `README.md` — minimal project readme.
- No build step, package manager, bundler, or test suite. There is no `package.json`.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There are no automated tests or linters configured. Verify changes by playing the game in a browser, including the touch controls when relevant.

## Code structure (`index.html`)

- **Markup** (~lines 1–421): `#container` wraps a `<canvas id="gameCanvas">`, the HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), boss bar, combo/stage banners, top-bar buttons (mute, reduced motion, pause), touch controls (`#touchControls`: joystick zone, BOMB and FIRE buttons), and the start/game-over `#overlay` with difficulty selection.
- **Script** (starts ~line 422):
  - Persistence keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`). Current reads/writes are not wrapped in try/catch; when adding new ones, wrap them so the game still works when storage is unavailable (private mode, blocked site data).
  - Config tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
  - Audio: lazily created `AudioContext` via `ensureAudio()`, with `beep()` and `noiseBurst()` helpers. Audio starts only after a user gesture.
  - Layout: logical play field is `BASE_W × BASE_H` (480 × 720), scaled by `resize()`.
  - Input: keyboard (`keys` map, arrows/WASD, Space fire, X/Shift bomb, P/Esc pause), mouse (steer and click to fire), and touch (joystick via `joystickStart/Move/End`, FIRE and BOMB buttons).
  - Game state: module-level `let` variables (`score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, etc.) reset by `resetGame()`.
  - Simulation: `update(dt)` runs each frame; spawning via `spawnEnemy()` / `spawnBoss()`; weapons via `fireWeapon()` and boss pattern helpers (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, etc.); power-ups via `maybeSpawnPowerUp()` / `applyPowerUp()`.
  - Rendering: `drawPlane()`, `drawBossShip()`, explosions and screen shake. `trackPerf()` lowers the `quality` level (0 full, 1 medium, 2 low) when frame times are poor.
  - Accessibility: `announce()` writes to the `#srLive` live region; reduced-motion mode disables shake and flash effects.

## Conventions

- Keep the game as a single self-contained file unless the user asks for a split. Do not add dependencies or a build step without discussion.
- Plain ES2015+ JavaScript, no frameworks. Match the existing style: 2-space indentation, single quotes, `const`/`let`.
- Keep the touch and keyboard paths in sync when changing controls.
- Use `localStorage` keys with the `skyfighter.` prefix for any new persisted setting.
- Keep the layout working at phone widths; the page uses `touch-action: none` and `overflow: hidden`, so don't introduce scrolling UI.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-un2858`). Do not push to other branches without permission.
- Do not open pull requests unless explicitly asked.
