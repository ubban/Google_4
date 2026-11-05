# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (inline `<style>` and one inline `<script>`). There are no build steps, no package manager, no bundler, and no external dependencies.

Other files:
- `README.md` — a one-line placeholder.
- `.git/` — the repository. Work happens on feature branches (see Git below).

## Running and testing

- To play, open `index.html` directly in a modern browser, or serve the directory with any static server (for example `python3 -m http.server`) and visit it.
- There is no test suite, linter, or build command configured. Verify changes by loading the page and playing: check the affected behavior in a real browser, and look at the browser console for errors.
- Keep the game working with mouse, keyboard, and touch input, and keep the layout usable on phone-sized screens.

## Code structure of `index.html`

The file is organized top to bottom as:

1. **`<head>` / `<style>`** — viewport and theme meta tags, a data-URI favicon, and all CSS. The HUD, overlays, and touch controls are styled here. `body.reduced-motion` is a class hook that disables motion effects.
2. **`<body>` markup** — the `#container` wrapper, the `#gameCanvas` canvas, the HUD (`#hud`, with stat chips for score, best, multiplier, weapon, hearts, bombs), the `#overlay` start/pause/game-over panels, the `#vignette` and `#flash` effect layers, touch controls (`#touchControls`, joystick, FIRE and BOMB buttons), and a hidden `#srLive` region for screen-reader announcements.
3. **`<script>`** — one IIFE containing all game logic:
   - **Config constants:** `DIFFICULTIES` (easy / normal / hard: spawn rate, enemy speed, enemy fire chance, lives, starting bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (background palettes, cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank, with `minStage` gating), and `BOSSES` (six boss designs, cycled by stage).
   - **Persistence:** `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. All reads and writes go through these keys; keep the prefix when adding new keys.
   - **Audio:** a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a `sfx` table. Sound is created only after a user gesture.
   - **Rendering:** `drawPlane`, `drawBossShip`, and `draw()`, using 2D canvas. The game uses a fixed logical resolution of `BASE_W` × `BASE_H` (480 × 720), scaled by `resize()`.
   - **Input:** keyboard (`keys` map; Arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses), mouse (`mouseTarget`), and touch (joystick, fire, and bomb buttons).
   - **Simulation:** `update(dt)` advances enemies, bullets, power-ups, particles, and bosses. `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, and `useBomb` handle spawning and combat. `resetGame()` restores state for a new run.
   - **Loop:** `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 50 ms, updates and draws only while running and not paused, and tracks frame timing for adaptive quality (`trackPerf`, `quality`).

Gameplay facts worth knowing before changing balance:
- Weapons: **VULCAN** (default) and **LASER**. Weapon level 1–3 controls spread and lanes.
- Power-ups: life (red), bomb (amber), and weapon (green).
- Stages advance after `KILLS_PER_STAGE` kills and end in a boss fight. The boss HP is `70 + stage × 35`, scaled by the difficulty and boss multipliers.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not introduce a build step or external dependencies without asking.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in JS, and small named functions grouped by concern as above.
- Use the existing DOM ids and config objects instead of hard-coding values in new places. Add new tunable numbers to `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` where they fit.
- Respect accessibility hooks already present: `prefers-reduced-motion`, the `reducedMotion` toggle, the `#srLive` announcer (`announce()`), and the touch-friendly controls.
- The existing `localStorage` calls are not wrapped in `try/catch`. When adding new storage access, guard it with `try/catch` so the game stays playable if storage is blocked (for example in private browsing).

## Git

- Develop on the branch named for the task (currently `claude/gifted-tesla-hptnbr`). Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages that explain the why as well as the what.
- Do not open a pull request unless explicitly asked.
