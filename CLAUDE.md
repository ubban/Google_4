# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file:

- `index.html` — markup, inline CSS (`<style>`), and inline JavaScript (`<script>`, wrapped in an IIFE with `'use strict'`). It is about 1,670 lines.
- `README.md` — a stub. Keep it short.

There is no build step, no bundler, no package manager, no framework, and no test suite. Do not add any of these unless the user asks. Open `index.html` directly in a browser to run the game.

## Working on the code

- **Keep it single-file.** Put new CSS in the `<style>` block and new JS in the existing `<script>` IIFE. Do not split files or add external dependencies or CDN scripts unless asked.
- **No global leaks.** Everything lives inside the IIFE. Keep it that way.
- **Preserve the existing style.** Use `const`/`let`, arrow functions for callbacks, two-space indentation, and single quotes for JS strings. Match the section comments (`// ---------- Input ----------`) when adding a new section.
- **Verify in a browser.** There are no automated tests, so after a gameplay or UI change, load `index.html` in a browser and play through the affected path. Check desktop (keyboard and mouse) and, for input or layout changes, touch-sized viewports.
- **Mind mobile.** The page disables pinch/zoom, uses `touch-action: none`, and has a touch joystick and on-screen FIRE/BOMB buttons. Any layout or input change must still work at phone width.

## Architecture map (`index.html`)

Roughly in file order:

| Area | Key identifiers | Notes |
|---|---|---|
| Markup | `#gameCanvas`, `#hud`, `#overlay`, `#touchControls`, `#vignette`, `#flash` | HUD stats are DOM elements, not canvas-drawn. |
| Styles | `:root { color-scheme: dark }`, `@keyframes heartBeat` | `.reduced-motion` on `<body>` disables motion effects. |
| Config | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` | Tuning data lives here. Prefer editing these tables over hard-coding numbers. |
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` | `localStorage` keys under the `skyfighter.*` prefix. Reads and writes are **not** wrapped in try/catch, so guard any new storage access that could throw (e.g. private mode). |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx`, `EXPLODE_SFX` | Web Audio. The context is created only after a user gesture (`ensureAudio()`). `beep` and `noiseBurst` return early when `muted`. |
| Layout | `BASE_W = 480`, `BASE_H = 720`, `resize()` | The canvas buffer is always 480×720. `resize()` sets only the CSS size, scaling up to 1.6× to fit the viewport. Draw in logical coordinates. |
| Input | `keys`, `keydown`/`keyup`, joystick functions, `canvasPointFromEvent` | Keyboard, mouse, and touch all feed the same `player` state. |
| Game state | `resetGame()`, `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` | `resetGame()` is the single place that resets a run. Stage progress lives in `stage`, `kills`, `stageKills`, `bossWarned`. |
| Spawning & combat | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `rectsOverlap`, `spawnExplosion` | Collision uses axis-aligned rectangles. |
| Weapons | `fireWeapon()` — `vulcan` and laser variants by `player.level` | Level 1–3 changes spread and lane count. |
| Bosses | `BOSS_PATTERNS`, `updateBoss`, `killBoss`, `fireRadial`, `fireCross`, etc. | Each pattern is a function that spawns bullets. Add new patterns to `BOSS_PATTERNS`. |
| Update | `update(dt)` | Fixed per-frame logic using `dt` in seconds. |
| Draw | `draw()`, `drawPlane`, `drawBossShip`, `drawClouds` | Pure rendering. Do not mutate game state here. |
| Loop | `loop(now)` | `dt` is clamped to 0.05 s. `trackPerf` adjusts a `quality` level (0–2) when frame times drift. |

### Game flow

Start overlay → `resetGame()` → `loop` calls `update(dt)` then `draw()` each frame while `running && !paused`.

Each stage needs `KILLS_PER_STAGE` (18) kills in `stageKills`. Reaching that sets `bossWarned` and spawns a boss about 1.2 s later via `spawnBoss()`. The boss design is `BOSSES[(stage - 1) % BOSSES.length]` and the background is `THEMES[(stage - 1) % THEMES.length]`. `killBoss()` increments `stage` and resets the stage counters. Losing all lives calls `loseLife()` → `endGame()`. The best score is saved when a run ends.

## Controls (keep the in-game hint in sync)

The hint text is at `index.html` near line 417 (`#controlsHint`). If you change a control, update that hint too.

- **Keyboard:** Arrows / WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses.
- **Mouse:** move to steer, click to fire.
- **Touch:** drag on the left to fly, tap FIRE, tap BOMB.

## Conventions

- Keep the game playable with both reduced motion and default motion. Gate new screen-shake or flashing effects behind `reducedMotion`.
- Keep announcements for screen readers going through `announce()` (the `#srLive` region).
- Do not add analytics, network calls, or third-party assets.
- Commit messages: short imperative subject line describing the gameplay or code change.

## Git

- Development branch for this work: `claude/gifted-tesla-5bpfy2`. Push with `git push -u origin claude/gifted-tesla-5bpfy2`.
- Do not open a pull request unless the user asks for one.
