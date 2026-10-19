# CLAUDE.md

Guidance for working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- `index.html` is the entire game: markup, CSS, and one inline `<script>` block.
- `README.md` is a placeholder with no project documentation.

There is no build step, package manager, bundler, test suite, or linter. To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Repository layout

```
index.html   # Full game: <style> (lines ~10-360), <body> HUD/overlays (~361-421), <script> (~422-end)
README.md    # Placeholder
```

Inside the script, everything lives in one IIFE with `'use strict'`. Sections are marked with `// ---------- Name ----------` comments. Key areas, in file order:

| Area | Notes |
| --- | --- |
| DOM refs and constants | `STORAGE_*` keys, `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE` |
| Adaptive performance | `trackPerf` drops `quality` (0/1/2) when frame time is high |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Web Audio is created lazily on first user gesture |
| Input | `keys` map (keyboard), joystick and FIRE/BOMB buttons (touch), mouse steering and click-to-fire on the canvas |
| Game state | `resetGame`, `update(dt)`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `endGame` |
| Enemies and bosses | `spawnEnemy`, `spawnBoss`, `BOSS_PATTERNS` (radial, spread, spiral, cross, homing, and so on), `updateBoss`, `killBoss` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Loop | `loop(now)` runs under `requestAnimationFrame`. `dt` is clamped to 0.05 s |

## Game mechanics to keep in mind

- **Stages:** Each stage needs `KILLS_PER_STAGE` (18) kills. Each stage has a theme from `THEMES` and ends with a boss from `BOSSES`, which cycles.
- **Enemy tiers:** `grunt` and `interceptor` appear from stage 1. `tank` appears from stage 2.
- **Difficulty:** `easy`, `normal`, and `hard` set spawn rate, enemy speed and fire chance, lives, starting bombs, and boss HP multiplier. Stored in `skyfighter.difficulty`.
- **Persistence:** The best score is stored in `localStorage` under `skyfighter.bestScore`. Mute (`skyfighter.muted`), difficulty, and reduced motion (`skyfighter.reducedMotion`) are also persisted.
- **Controls:** Arrows/WASD move, Space or hold-to-fire, X or Shift for bomb, P or Esc to pause. Mouse steers and clicks to fire.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and `camelCase` for functions and variables.
- Keep new constants with the other tuning tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) so balance changes stay in one place.
- Persisted keys use the `skyfighter.` prefix. Wrap new `localStorage` access in the same kind of guard the file already uses, because storage can be unavailable.
- Accessibility matters here. Announce important state changes through `announce()` (the `srLive` region), and gate screen shake, flashes, and pulses behind `reducedMotion`.
- Keep the game self-contained. Do not add external scripts, fonts, or assets unless the user asks for them.

## Verifying changes

There is no automated test or lint setup. After a change:

1. Open `index.html` in a browser and check the console for errors.
2. Play through at least one stage, a boss fight, a bomb, a pause/resume, and a game over. Confirm the best score persists after reload.
3. Check the mobile layout (touch controls) with browser device emulation if the change touches HUD, overlays, or touch handling.

## Git workflow

- Develop on the branch the task names. Do not push to `main` unless told to.
- Keep commits focused. Describe gameplay or UI changes in the commit message in plain terms.
