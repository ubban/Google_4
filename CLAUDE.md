# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single HTML5 canvas page.

- `index.html` — the entire game: HTML markup, CSS (in a `<style>` block), and JavaScript (one IIFE in a `<script>` block near the end of the file).
- `README.md` — placeholder; currently contains only the title.

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not add these unless the user asks.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- Verify changes by playing in a browser. There are no automated tests to run.

## Code layout inside `index.html`

- **Markup** (`<body>`): `#container` holds `#gameCanvas`, the HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), boss bar, combo/stage banners, touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **Styles**: the `<style>` block in `<head>` uses a dark color scheme and a `#0b1d3a` theme color. Mobile layout relies on `touch-action: none` and safe-area insets.
- **Script config constants** (top of the IIFE):
  - `DIFFICULTIES` — `easy` / `normal` / `hard` tuning (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier).
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — per-stage sky/cloud/sun palettes (cycled by stage).
  - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank` (HP, speed, score, minimum stage).
  - `BOSSES` — boss names, colors, and HP multipliers; `BOSS_PATTERNS` holds the bullet patterns.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes are not wrapped in try/catch, so any change here should keep private-mode and blocked-storage behavior in mind.
- **Game loop**: `update(dt)` advances game state, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`.
- **Weapons and power-ups**: `fireWeapon()` and `applyPowerUp()`. Weapons are `vulcan` and `laser`, each with 3 levels; a weapon power-up raises the level and cycles weapon type once maxed.
- **Bombs**: `useBomb()` clears the screen and damages bosses. Bomb count is capped by difficulty.
- **Audio**: a Web Audio synth (`beep`, `noiseBurst`, and the `sfx` table). Audio starts lazily via `ensureAudio()` on first user gesture.
- **Input**: keyboard (arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause), mouse (move to steer, click to fire), and touch (joystick on the left, FIRE/BOMB buttons on the right).
- **Accessibility**: `prefers-reduced-motion` is honored by default, with a user toggle (`#motionBtn`) that disables screen shake, flashes, and pulses. A visually hidden `#srLive` region announces events via `announce()`.

## Conventions

- Keep the game as a single self-contained file unless the user asks for a restructure. Don't introduce external dependencies or CDN scripts.
- Preserve the existing style: 2-space indentation, `const`/`let`, and small named helper functions.
- Keep the canvas base resolution (`BASE_W` 480 × `BASE_H` 720) in mind when changing positions or sizes; `resize()` scales the canvas to the viewport.
- When changing gameplay numbers, tune them in the config constants rather than hard-coding values in the update loop.
- Any new UI control needs keyboard and touch reachability, and an accessible label or `title`.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-6q9tdc`). Commit with clear messages and push with `git push -u origin <branch>`.
- Do not open pull requests unless the user explicitly asks.
