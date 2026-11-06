# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser.
The entire game is a single self-contained file, `index.html` (~1,670 lines):
inline `<style>`, the markup for the HUD/overlays/touch controls, and one inline
`<script>` (an IIFE) that holds all game logic. There is no build step, no
bundler, no `package.json`, and no test suite. `README.md` is a placeholder.

## Running it

- Open `index.html` directly in a browser, or serve the directory with any static
  server (e.g. `python3 -m http.server` and visit `/index.html`).
- There are no dependencies to install and no lint/test/build commands to run.
  Verify changes by loading the page and playing, or with a headless browser
  (Playwright/Chromium is available in the cloud environment).

## Code layout (inside `index.html`)

- **`<style>`** — dark color scheme, HUD pills (`#hud .stat`), overlays, touch
  controls, and the vignette/flash effects. The canvas is scaled with `resize()`
  to fit the viewport, preserving a logical resolution of `BASE_W=480 × BASE_H=720`.
- **`<body>`** — `#gameCanvas` plus HUD elements (score, best, multiplier,
  weapon, hearts, bombs), the boss bar, the top bar (mute / reduced-motion /
  pause buttons), touch controls (`#moveZone` joystick, FIRE, BOMB), and the
  start/pause/game-over overlay (`#overlay`, difficulty buttons `.diffBtn`).
- **`<script>`** — in this order, roughly:
  1. DOM references and `STORAGE_*` keys.
  2. Tunables: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (six
     background palettes cycled by stage), `ENEMY_TYPES`, `BOSSES` (six bosses
     cycled by stage, each with an `hpMul`).
  3. Persistent settings via `localStorage` (best score, mute, difficulty,
     reduced motion).
  4. Audio: a lazily created `AudioContext` (`ensureAudio`) with `beep`,
     `noiseBurst`, and a named `sfx` table. Nothing plays until a user gesture.
  5. Input: keyboard (`keys` map, `Space`, `KeyP`/`Escape` pause, `KeyX`/`Shift`
     bomb), mouse (steer and click to fire on the canvas), and touch (joystick,
     FIRE, BOMB).
  6. Game state (`score`, `lives`, `bombCount`, `stage`, `stageKills`, `boss`,
     arrays for bullets/enemies/powerUps/particles), plus `resetGame()`.
  7. Spawning (`spawnEnemy`, `spawnBoss`), weapons (`fireWeapon`), bombs
     (`useBomb`), power-up effects (`applyPowerUp`), and `update(dt)` /
     `updateBoss(dt)` for simulation.
  8. Rendering: `draw()` and helpers (`drawPlane`, `drawBossShip`,
     `drawClouds`), plus the adaptive `quality` level from `trackPerf`.
  9. `loop(now)` — a `requestAnimationFrame` loop with `dt` clamped to 50 ms.

## Game rules worth knowing before changing anything

- **Stages:** a stage ends after `KILLS_PER_STAGE` (18) kills. Then a boss
  spawns (after a 1.2 s delay). Defeating the boss advances `stage`, shows a
  banner, and changes theme and boss by `(stage - 1) % length`.
- **Enemies:** `grunt` (1 HP, minStage 1), `interceptor` (fast, weaving,
  minStage 1), `tank` (4 HP, minStage 2).
- **Difficulty** (`DIFFICULTIES`) controls spawn cadence, enemy speed, shot
  chance, starting lives and bombs, and boss HP multiplier. Easy/Normal/Hard.
- **Weapons:** `vulcan` (spread shot) and `laser` (pierce). Each has level 1–3.
  A weapon power-up raises the level; past level 3 it switches weapon type and
  resets the level to 1.
- **Power-ups** dropped by enemies: `life`, `bomb`, `weapon`. They fall slowly
  and are collected by touching the player.
- **Score/multiplier:** kills feed a combo chain; losing a life breaks it
  (`breakChain`). A boss kill is worth 500 points.
- **Damage:** a hit calls `loseLife()`, which grants 1.5 s of invulnerability.
  Collision with a boss body also costs a life.
- **Best score** is saved to `localStorage` under `skyfighter.bestScore`.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it.
  If a split is ever made, update this file.
- Keep game constants in the tunables block near the top of the `<script>`
  rather than scattering magic numbers through update/draw code.
- Preserve the mobile layout: keep `touch-action: none` on the game surface and
  `env(safe-area-inset-*)` for the HUD. Test touch and keyboard/mouse input
  paths when changing controls.
- Respect the reduced-motion toggle (`reducedMotion`) for any new screen shake,
  flash, or pulse effect.
- Audio must start from a user gesture (`ensureAudio()`), and mute state must
  be respected by any new `sfx` call.
- Accessibility: announce important state changes through `announce()` (the
  `#srLive` live region).
- Don't introduce external network dependencies or CDN scripts; the game is
  meant to work as a single offline file.
- Stay with plain HTML/CSS/JS: no frameworks, no transpilation, no build step.

## Git workflow

- Develop on the branch assigned to the session (currently
  `claude/gifted-tesla-u4eca1`). Push with `git push -u origin <branch>`.
- Write descriptive commit messages. Do not open a pull request unless asked.

## Known gaps

- No automated tests, linter, or CI. Gameplay changes should be verified by
  playing the game in a browser.
- `README.md` contains only a title and a stray timestamp line; it is not
  documentation for the game.
