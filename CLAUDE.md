# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style
vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

- The whole game lives in a single file: `index.html` (~1,670 lines).
  It contains the HTML markup, inline `<style>`, and one inline `<script>` IIFE.
- `README.md` is a placeholder and has no setup instructions.
- There is no build step, package manager, test suite, or linter configured.

## Running the game

No install is needed. Open the file directly or serve the directory statically:

```bash
# any static server works, e.g.
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

Browser APIs in use include `localStorage`, `AudioContext` (Web Audio), `matchMedia`,
pointer/touch events, and `requestAnimationFrame`. Test in a real browser; there is no
headless test harness.

## Code layout (inside `index.html`)

Key sections of the inline script, roughly in file order:

| Area | What it holds |
| --- | --- |
| DOM refs | `getElementById` lookups at the top of the IIFE |
| Storage keys | `skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion` |
| Adaptive quality | `trackPerf` lowers/raises a `quality` level from frame timings |
| Config tables | `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`; sound effects are synthesized, no audio files |
| HUD / feedback | hearts, bombs, combo pop, stage banner, screen shake, flash, vignette |
| Input | keyboard `keydown`/`keyup` handlers, touch joystick + fire/bomb buttons, mouse fire |
| Game state | `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, power-ups, bombs |
| Boss patterns | `BOSS_PATTERNS` and the `fire*` helpers they call |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Loop | `loop(now)` drives `update` and `draw` each frame |

When changing gameplay numbers, prefer editing the config tables (`DIFFICULTIES`,
`ENEMY_TYPES`, `BOSSES`, `THEMES`) over hard-coding values in logic.

## Controls (for reference when testing)

- Move: arrow keys or `WASD`
- Fire: `Space` (hold), or mouse-hold on the fire button
- Bomb: `X` or `Shift`
- Pause: `P` or `Esc`
- Touch devices: on-screen joystick (left), fire and bomb buttons

## Conventions

- Keep the game self-contained in `index.html` unless there is a clear reason to split
  it. Do not add external CDN scripts or a bundler without asking.
- Keep the existing style: `'use strict'` IIFE, `const`/`let`, two-space indentation,
  single quotes in JS.
- Preserve accessibility hooks: the `srLive` live region (`announce()`) and reduced-motion
  support (`reducedMotion`, `prefers-reduced-motion`).
- Preserve mobile behavior: the viewport meta, `touch-action`, and the touch control
  layout. Verify changes at phone width.
- Persisted values go through the `skyfighter.*` localStorage keys. Wrap new storage
  access in try/catch, since storage can be unavailable (private windows, blocked
  site data).
- The `AudioContext` is created lazily by `ensureAudio()`, which is called from user-gesture
  handlers. Keep new audio triggers going through it.

## Making changes

- Read the relevant section of `index.html` before editing; the file is large and
  sections depend on shared state in the IIFE.
- Test gameplay changes by actually playing: start, clear a stage, reach a boss,
  lose a life, game over, and check the best-score update.
- Keep commits focused. Git branch rules for this environment are set by the session,
  not by this file.
