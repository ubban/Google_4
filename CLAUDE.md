# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the whole game: inline `<style>`, markup, and one inline `<script>` (~1,670 lines). No build step, no bundler, no dependencies.
- `README.md` — minimal placeholder.

To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Repository layout

```
.
├── index.html   # Entire game (HTML + CSS + JS)
├── README.md
└── CLAUDE.md    # This file
```

Keep the game as a single file unless there is a clear reason to split it. Do not introduce a build system, package manager, or external runtime dependencies without asking first.

## Game architecture (inside `index.html`)

- **Rendering:** a single `<canvas id="gameCanvas">` drawn each frame by `draw()`; game logic advances in `update(dt)` (delta-time based). Boss movement lives in `updateBoss(dt)`; boss attack patterns are in `BOSS_PATTERNS`, and boss visuals in `drawBossShip(b)`.
- **Entities:** the player is `player` (weapon `'vulcan'` or `'laser'`, `level` 1–3, `invuln`, `cooldown`). Enemies are created in `spawnEnemy()`, bosses in `spawnBoss()`, and explosions in `spawnExplosion()`.
- **Stages:** `stage` advances after `KILLS_PER_STAGE` (18) kills, which triggers a boss via `bossWarned`. Stage banners use `showStageBanner()`. `BOSSES` holds per-stage boss designs.
- **Difficulty:** the `DIFFICULTY` table (`easy` / `normal` / `hard`) controls spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs, and a `bossHpMul` multiplier. Keep these tables as the single source of tuning values.
- **Audio:** Web Audio API, created lazily through `ensureAudio()` after a user gesture. Respect the mute toggle.
- **HUD and overlays:** DOM elements (`#hud`, `#weaponStat`, `#bombs`, `#stageBanner`, overlay card) sit above the canvas. Update them through the existing helpers (`renderBombs`, `weaponLabel`, etc.) rather than duplicating logic.
- **Input:** keyboard (arrows/WASD move, Space fire, X or Shift bomb, P/Esc pause), mouse (move to steer, click to fire), and touch (drag left zone to fly, FIRE and BOMB buttons). Touch listeners use `{ passive: false }` with `preventDefault()`.

## Persistence

Settings and scores are stored in `localStorage` under these keys. Do not rename them without a migration:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `'easy'`, `'normal'`, or `'hard'` |
| `skyfighter.reducedMotion` | `'1'` or `'0'` (absent means use the OS preference) |

Wrap `localStorage` access so the game still runs if storage is unavailable (private mode, blocked site data).

## Accessibility and motion

- A reduced-motion mode exists (`body.reduced-motion`) that disables screen shake and flash effects. New visual effects that flash or shake must respect it.
- The page must work at phone width with no horizontal scroll; the viewport meta disables zoom and the layout uses `env(safe-area-inset-*)`. Keep mobile touch controls working when changing input code.

## Working conventions

- **Edit surgically.** Because everything is in one file, prefer small, targeted edits over rewriting large sections.
- **Preserve tuning values.** Changes to difficulty numbers, boss HP, or spawn timing should be deliberate and called out in the commit message.
- **No test suite or linter is configured.** Verify changes by loading `index.html` in a browser and playing through at least one stage, including a boss fight, on desktop and in a mobile-sized viewport. Check the console for errors.
- **Keep the code style consistent with the existing file:** 2-space indentation, `const`/`let`, plain ES2015+ JavaScript, no frameworks.

## Git

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-ju9pq2`), not directly on `main`.
- Write clear, descriptive commit messages that explain *why* a change was made.
- Do not open a pull request unless the user explicitly asks for one.
