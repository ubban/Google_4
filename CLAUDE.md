# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines): all HTML, CSS (in a `<style>` block), and JavaScript (in one IIFE `<script>` block, `'use strict'`). There is no build step, no bundler, no package manager, and no external dependencies.
- `README.md`: a minimal placeholder.

There are no tests, linters, or CI configured. Verify changes by opening `index.html` in a browser (or serving it with any static server) and playing.

## Running the game

```bash
# Any of these works; no install needed
open index.html                 # macOS
xdg-open index.html             # Linux
python3 -m http.server 8000     # then visit http://localhost:8000
```

## Code layout inside `index.html`

- **Markup** (`<body>`, ~line 360+): `#gameCanvas`, `#hud` (score, best, multiplier, weapon, hearts, bombs), `#overlay` (title/game-over card with difficulty buttons and `#startBtn`), `#stageBanner`, `#comboPop`, `#vignette`, `#flash`, `#bossBarWrap`, touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and icon buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`).
- **Styles** (`<style>`, ~lines 10–360): colors and layout are inline. `body.reduced-motion` disables shake, flashes, and pulses.
- **Script** (`<script>`, ~line 422 onward), in roughly this order:
  - DOM references and `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
  - Adaptive performance (`trackPerf`, `quality` 0–2) that lowers effect quality when frame time is high.
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - Audio: `sfx` built on the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`). Audio starts only after a user gesture.
  - Rendering helpers (`drawPlane`, `drawBossShip`, `drawClouds`) and the main `draw()`.
  - Game logic: `update(dt)`, `fireWeapon`, boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `useBomb`, `applyPowerUp`.
  - Input: keyboard (`keys[e.code]`), touch joystick, mouse, and device-motion handling.
  - `loop(now)` runs on `requestAnimationFrame` with a variable `dt` clamped to 50 ms, then calls `trackPerf`, `update(dt)`, and `draw()` while running and not paused. `startBtn` starts a run.

## Game design facts (keep in sync with code)

- **Difficulties** (`easy` / `normal` / `hard`) set spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. The selection is persisted in `localStorage`.
- **Stages**: 18 kills (`KILLS_PER_STAGE`) trigger a boss. Bosses cycle through `BOSSES` by stage index. Stage themes cycle through `THEMES`.
- **Enemies**: `grunt` (stage 1+), `interceptor` (stage 1+, weaves), `tank` (stage 2+, 4 HP).
- **Weapons**: `vulcan` (spread grows with level) and `laser`. Power-ups include `weapon`, `bomb`, and `life`.
- **Scoring**: base scores per enemy type, combined with a chain multiplier that `breakChain()` resets.
- Best score is saved to `localStorage` when a run ends with a new high score.

## Conventions

- Keep the game as a **single file** unless the user asks for a split. Do not add a build system or npm dependencies without a clear request.
- Preserve the IIFE and `'use strict'`. Keep new state inside the IIFE rather than on `window`.
- Put tuning numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) instead of hardcoding them in logic.
- Any new `localStorage` access should use a `STORAGE_*` constant with the `skyfighter.` prefix. Keep reads and writes defensive, since storage can be blocked.
- Respect `reducedMotion`: new screen shake, flashes, or pulsing effects should be gated on it and added to the `body.reduced-motion` CSS list.
- Announce important state changes through `announce()` for screen readers. Keep the `#srLive` region in use.
- Keep touch and keyboard controls working together. Any new action needs both a keyboard binding and a touch/button path if it matters for play.
- Maintain phone-width layout (viewport meta, `touch-action: none`, safe-area insets) and avoid horizontal scroll.

## Working with this repo

- Default branch: `main`. Feature work is done on branches named `claude/...`.
- Commit with clear messages describing the gameplay or code change. Do not push or open PRs unless asked.
- Before committing, open the game in a browser and confirm it loads without console errors and that the affected feature works on both keyboard and touch layouts.
