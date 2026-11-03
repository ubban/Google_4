# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up written as a single-page HTML5 canvas game. `README.md` is a placeholder and does not describe the project.

## Repository layout

```
.
├── index.html   # The entire game: markup, CSS (<style>), and JS (<script>) in one file (~1,670 lines)
├── README.md    # Placeholder
└── CLAUDE.md    # This file
```

There is no build system, package manager, test suite, linter, or CI configuration. Do not add one unless asked.

## Running the game

No build step is needed. Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`) and visit it. Changes take effect on reload.

## How `index.html` is organized

- **`<style>`** (around lines 10–360): all styling, including the HUD pills, hearts/bombs, the glassmorphic overlay card, touch controls, and the `.reduced-motion` class.
- **Markup** (around lines 360–420): the `<canvas id="gameCanvas">`, HUD elements, overlay, touch controls, and the control hint text.
- **`<script>`** (starts around line 422): a single IIFE (`'use strict'`) containing all game logic. Major sections, roughly in file order:
  - DOM references and storage keys (`STORAGE_*`).
  - Adaptive performance (`trackPerf`, `quality` 0–2).
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is synthesized with WebAudio, not loaded from files.
  - HUD helpers and effects (hearts, bombs, combo popups, stage banner, screen shake, hit flash).
  - Input: keyboard listeners, pointer/mouse, and touch joystick plus FIRE/BOMB buttons.
  - Game state: `resetGame`, `update(dt)`, `draw()`, `loop(now)`.
  - Entities and combat: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `endGame`, `togglePause`.
  - Boss attack patterns: `BOSS_PATTERNS` and the `fire*` helpers, driven by `updateBoss`.
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`.

Persistence uses `localStorage` under the keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Wrap any new storage access in try/catch, because storage can be unavailable (private windows, blocked site data).

## Controls (keep the hint text in sync)

The on-screen hint at `index.html` (`#controlsHint`) lists these. If you change a binding, update the hint too.

| Action | Keyboard | Mouse / touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Mouse move; touch: drag left zone (joystick) |
| Fire | Space (hold) | Hold mouse button; touch: FIRE button |
| Bomb | X or Shift | BOMB button |
| Pause | P or Esc | Pause button |

## Conventions when editing

- **Keep it a single file.** The project is intentionally self-contained. Do not split out modules, add a bundler, or add external dependencies or CDN scripts unless the user asks.
- **Tune through the tables.** Balance changes (enemy stats, boss HP multipliers, spawn rates, lives/bombs per difficulty) belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`, not as inline magic numbers.
- **Time-based updates.** `update(dt)` receives seconds, capped at 0.05 per frame. Express movement and timers in `dt`, not per frame.
- **Respect reduced motion.** Effects such as screen shake and flashes should check `reducedMotion` (or the `.reduced-motion` class).
- **Accessibility.** Announcements for screen readers go through `announce()` into the `#srLive` region. Keep new state changes announced when they matter (game over, stage change, new best).
- **Mobile.** The game scales to a 480×720 base via `resize()`. Verify layout and touch controls at phone width when changing HUD or overlay CSS.
- **Keep the game loop responsive.** Avoid allocating large arrays or doing heavy work per frame in `update`/`draw`; the adaptive `quality` setting exists for slow devices.

## Verifying changes

There are no automated tests. For gameplay or UI changes, load the page in a browser and check the affected flow manually: start, play, pause/resume, bomb, a stage transition, boss fight, game over, and the best-score update. Check both keyboard and touch layouts when relevant.

## Git

- Develop on the branch the task designates (currently `claude/gifted-tesla-c8w2tn`). Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood, matching the existing history (for example, "Add Sky Fighter: ...").
- Do not open a pull request unless the user explicitly asks for one.
