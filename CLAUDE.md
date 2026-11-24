# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines): HTML markup, inline CSS (`<style>`), and one inline `<script>` IIFE containing all game logic, rendering, audio, and input.
- `README.md`: a minimal placeholder. Currently it holds only a title and a stray timestamp line.

There is no build system, no package manager, no test suite, and no external dependencies. The game uses a 2D `<canvas>` and the Web Audio API.

## Running the game

No build step is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code layout (inside `index.html`)

The inline script is wrapped in `(function () { 'use strict'; ... })()`. Rough sections, in file order:

| Area | Notes |
| --- | --- |
| DOM refs | `document.getElementById` lookups at the top of the script. IDs must match the markup. |
| Storage keys | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all `localStorage`). |
| Config tables | `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`. Tune balance here. |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio starts lazily on the first user gesture. |
| Adaptive quality | `trackPerf` lowers the `quality` level (0 full, 1 medium, 2 low) when frame time is high. |
| UI feedback | Hearts/bombs HUD, combo pop, stage banner, screen shake, hit flash. |
| Input | Keyboard, touch joystick plus fire/bomb buttons, and mouse drag on the canvas. |
| Game state | `resetGame`, `update(dt)`, `draw()`, and `loop(now)`. |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`. |
| Weapons | `fireWeapon` (Vulcan and Laser, each with power levels), `useBomb`, and `applyPowerUp`. |
| Boss | `BOSS_PATTERNS` (bullet-pattern functions), `updateBoss`, `killBoss`. |
| Flow | `loseLife`, `endGame`, `togglePause`, and the start button handler. |

Keep the existing pattern of one shared `update(dt)` / `draw()` pair driven by `requestAnimationFrame` (`loop`). `dt` is measured per frame and clamped to 0.05 s, so long stalls do not cause large jumps.

## Gameplay facts to preserve

- **Controls:** mouse moves the ship and click fires. Arrows or WASD move. Space fires (held). `X` or Shift bombs. `P` or Escape pauses. On touch devices, drag the left zone to fly and tap FIRE/BOMB. The on-screen hint in `index.html` (`#controlsHint`) should stay in sync with these.
- **Difficulty:** `easy`, `normal` (default), `hard`. Each sets spawn rates, enemy shoot chance, lives, starting bombs, and boss HP multiplier.
- **Power-ups:** `life`, `bomb`, and `weapon` (toggles Vulcan/Laser and raises weapon level). Life is capped at `maxLives`.
- **Scoring:** a kill-chain multiplier. `breakChain` resets the chain and multiplier to 1.
- **Stages:** every `KILLS_PER_STAGE` kills advances the stage, which changes the theme and eventually spawns a boss. Bosses cycle through `BOSSES`.
- **Persistence:** best score, mute, difficulty, and reduced-motion preference persist in `localStorage`. Every read and write must tolerate storage being unavailable.

## Conventions

- **Keep it one file.** Do not split `index.html` into modules or add a bundler unless the user asks. The single-file, no-build design is intentional.
- **No dependencies.** Do not add libraries or CDN scripts without an explicit request.
- **Accessibility:** respect `prefers-reduced-motion` (see `reducedMotion` and the `reduced-motion` body class). Keep the `srLive` live region updated for important announcements. Keep ARIA labels on buttons.
- **Mobile:** keep the layout usable at phone widths and touch controls working. `BASE_W` / `BASE_H` (480 x 720) define the logical canvas size, and `resize()` scales it to fit.
- **Style:** match the existing code style: 2-space indent, single quotes, `const` by default, arrow functions for callbacks, and section comments using `// ---------- Name ----------`.
- **Naming:** constants are `UPPER_SNAKE_CASE`. Storage keys use the `STORAGE_*` prefix and the `skyfighter.` namespace.

## Working in this repo

- Verify changes by running the game in a browser and playing through at least one stage transition and one boss fight when touching gameplay code. Syntax-only checks are not enough.
- When changing balance numbers, update the relevant entry in `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` rather than hard-coding values in logic.
- Do not commit unrelated edits. Keep commits focused.
- Do not add a pull request unless the user explicitly asks for one.
