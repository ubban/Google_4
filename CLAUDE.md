# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire app lives in one file, `index.html` (inline CSS and JavaScript, no build step, no external dependencies).

Other files:
- `README.md` — minimal project readme.

There is no `package.json`, bundler, test suite, or linter configured.

## Running the game

- Open `index.html` directly in a browser, or serve the folder:
  `python3 -m http.server 8000` then visit `http://localhost:8000`.
- No install step is required.

## Code layout (`index.html`)

- `<style>` (around line 10): all CSS. Colors and HUD styling are inline; the HUD is a row of pill-shaped `.stat` elements, plus `#vignette`, `#flash`, boss bar, touch controls, and the start/pause overlay.
- `<body>` markup (after the style block): canvas `#gameCanvas`, HUD, overlay card with difficulty buttons (`.diffBtn`, `data-diff`), `#startBtn`, icon buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), and touch controls.
- `<script>` (around line 422): one IIFE holding all game logic. Key sections, roughly in file order:
  - DOM references and `STORAGE_*` keys (`skyfighter.*` in `localStorage`).
  - Config tables: `DIFFICULTIES` (easy/normal/hard), `THEMES` (sky palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, one per stage cycle), `KILLS_PER_STAGE`.
  - Audio: lazily created `AudioContext` via `ensureAudio()`, with `beep()`, `noiseBurst()`, and the `sfx` table. Mute state is persisted.
  - HUD helpers: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`/`flashBomb`.
  - Responsive scaling: `resize()` fits a fixed `BASE_W × BASE_H` (480×720) logical canvas into the viewport, capped at 1.6×.
  - Input: keyboard (`keys` map), mouse (`mouseActive`, `mouseTarget`), and touch (virtual joystick in `moveZone`, `fireBtn`, `bombBtn`).
  - Simulation: `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `spawnExplosion()`, `useBomb()`, `update(dt)`, `updateBoss(dt)`, `togglePause()`.
  - Rendering: `drawPlane()`, `drawBossShip()`, `drawClouds()`, `draw()`.
  - Main loop: `loop(now)` driven by `requestAnimationFrame`, with adaptive quality (`quality` 0/1/2) driven by `trackPerf()`.

## Controls (for reference when changing input code)

- Keyboard: Arrows/WASD move, Space fires, X or Shift triggers a bomb, P or Esc pauses.
- Mouse: move to steer, hold click to fire (`fireBtn` / canvas).
- Touch: drag left side to fly with the virtual joystick, tap FIRE, tap BOMB.

## Game-design conventions

- Difficulty changes spawn rate, enemy speed, enemy fire chance, starting lives/bombs, and boss HP multiplier. Adjust values in `DIFFICULTIES` rather than hard-coding them elsewhere.
- Each stage lasts `KILLS_PER_STAGE` kills, then a boss spawns. Theme is `THEMES[(stage-1) % THEMES.length]` and boss is `BOSSES[(stage-1) % BOSSES.length]`.
- Rendering quality (`quality` 0/1/2) is lowered automatically by `trackPerf()` when frame times are poor; keep expensive effects gated on it.
- Enemies and bosses are data-driven from `ENEMY_TYPES` and `BOSSES`; add new entries there.
- Persistent state (best score, mute, difficulty, reduced motion) uses `localStorage` with the `skyfighter.*` prefix. Wrap storage access so the game still works if storage is unavailable.
- Accessibility: respect `prefers-reduced-motion` (the motion toggle controls screen shake, flashes, and pulses). Keep the `#srLive` live region updated through `announce()` for important state changes.
- Visuals are drawn with the 2D canvas API; keep draw code inside `draw()` / helper draw functions and avoid per-frame allocations in hot paths.

## Conventions for edits

- Keep everything in `index.html` unless there is a clear reason to split files. Do not introduce a build step or frameworks without asking.
- Preserve the single IIFE structure and the mobile layout (`viewport-fit=cover`, `touch-action: none`, safe-area insets).
- When changing gameplay numbers, prefer editing the config tables at the top of the script.
- Verify changes by loading the page in a browser and playing through at least one stage, including pause, bomb, and game-over flows, on desktop and a narrow/mobile viewport.

## Git

- Develop and push on the feature branch you were given (currently `claude/gifted-tesla-qbyonj`), using `git push -u origin <branch>`. Do not push to `main` without explicit permission.
- Write descriptive commit messages in the imperative mood.
