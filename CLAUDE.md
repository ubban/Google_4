# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- Everything lives in `index.html`: markup, CSS (`<style>`, roughly lines 10–362), and one inline IIFE script (`<script>`, roughly lines 422–1668).
- There is no build step, no bundler, no `package.json`, and no test suite. Runtime dependencies are zero; nothing is loaded from a CDN.
- `README.md` is a placeholder and carries no project documentation.

To run it, open `index.html` in a browser, or serve the directory with any static file server.

## Repository layout

```
index.html   # The entire game (HTML + CSS + JS)
README.md    # Placeholder
CLAUDE.md    # This file
```

## Code structure of `index.html`

The script is one `(function () { 'use strict'; ... })()` closure. Keep new code inside it so state stays private.

Within the closure, the code is grouped roughly in this order:

1. **DOM refs**: `const xxxEl = document.getElementById(...)`. Add new HUD elements here.
2. **Storage keys**: `STORAGE_*` constants for `localStorage` (`skyfighter.*`).
3. **Config tables**: `DIFFICULTIES`, `THEMES` (stage backdrops), `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE`.
4. **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table). No audio files.
5. **HUD and effects**: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, and similar.
6. **Input**: keyboard (`keys` map), touch joystick and buttons, and mouse. Input handlers set state; `update()` reads it.
7. **Game state and reset**: `resetGame()`.
8. **Drawing**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
9. **Simulation**: `update(dt)` drives the per-frame logic. Power-ups, weapons (`fireWeapon`), boss patterns (`BOSS_PATTERNS`), and bosses (`updateBoss`, `killBoss`) all live here.
10. **Game flow**: `loseLife`, `endGame`, `togglePause`.
11. **Main loop**: `loop(now)` calls `trackPerf`, then `update` and `draw`, then requests the next frame. Start-button wiring is at the bottom.

## Key mechanics and constants

- **Canvas**: fixed logical size `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the CSS size to fit the window, capped at 1.6×.
- **Timestep**: `dt` is clamped to 50 ms. Keep game speeds expressed per second, not per frame.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Stages**: once `KILLS_PER_STAGE` (18) kills are reached, a boss spawns 1.2 s later. Defeating it advances to the next stage. Each stage uses a `THEMES` entry (indexed mod 6).
- **Adaptive quality**: `trackPerf` watches a 45-frame rolling average and steps `quality` between 0 (full), 1 (medium), and 2 (low) to hold framerate.
- **Persistence**: best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under `skyfighter.*`. Reads and writes use plain `localStorage` calls. Keep them defensive if you add more.
- **Accessibility**: `prefers-reduced-motion` seeds the reduced-motion toggle. Screen-reader text goes through `announce()` into the `srLive` live region.

## Controls

| Action | Keyboard | Touch |
|---|---|---|
| Move | Arrow keys or WASD | Left joystick zone |
| Fire | Space (hold) | Fire button (hold) |
| Bomb | X or Shift | Bomb button |
| Pause | P or Esc | Pause button |

Mouse: moving over the canvas steers the ship; mouse-down fires.

## Conventions

- **Match the existing style**: 2-space indentation, single quotes in JS, `const` by default, arrow callbacks, and short helper functions named in camelCase. Constants are UPPER_SNAKE.
- **Comments**: section banners use `// ---------- Name ----------`. Keep comments brief and explain *why*, not *what*.
- **Single file**: do not split `index.html` into modules or add a build step unless the user asks for it. The single-file, zero-dependency design is intentional.
- **No new external dependencies or CDN links** without asking. The game is meant to run offline from one file.
- **Per-frame work**: avoid allocating large arrays or objects in hot paths (`update`, `draw`). Prefer reusing or pooling; the adaptive quality system exists because frame time matters.
- **Mobile**: any new interaction must work on touch as well as keyboard. Preserve `preventDefault` on touch handlers that use `{ passive: false }`.

## Working on this repo

- Test changes by loading `index.html` in a browser and playing a few stages. Check all three difficulties, a boss fight, pause and resume, and game over.
- Check the layout at phone width as well as desktop. The canvas scales, so overlays and the touch controls must stay reachable.
- When changing balance numbers, adjust `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` rather than hard-coding values in the update loop.
- Git: develop on the branch you were assigned (currently `claude/gifted-tesla-a7slaa`), and do not push to `main` without explicit permission.
