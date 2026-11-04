# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- The entire game (HTML, CSS, and JavaScript) lives in `index.html`. There is no build step, no bundler, no package manager, and no test suite.
- `README.md` is a placeholder with no project documentation.
- To run the game, open `index.html` in a browser, or serve the directory with any static file server.

## Repository layout

```
index.html   # The whole game: <style> (~lines 10-361), HTML markup (~362-421), <script> (~422-1669)
README.md    # Placeholder
CLAUDE.md    # This file
```

Inside `index.html`, the script is one IIFE (`(function () { 'use strict'; ... })();`). Nothing is exported to `window`, so all game state is local to that closure.

## Code map (`index.html` script)

Roughly in file order:

| Area | Key names | Notes |
|---|---|---|
| DOM refs | `canvas`, `ctx`, `overlay`, `hud` elements | Grabbed once at the top of the IIFE |
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` | `localStorage` keys prefixed `skyfighter.` |
| Difficulty | `DIFFICULTIES` (easy / normal / hard) | Spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier |
| Stage config | `KILLS_PER_STAGE`, `THEMES` (sky palettes), `ENEMY_TYPES`, `BOSSES` | Stage themes cycle; bosses have HP multipliers and colors |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX` | Web Audio synthesis; no audio files |
| Input | `keys` map, `joystickStart/Move/End`, `canvasPointFromEvent`, touch and mouse listeners | Keyboard, mouse, and touch are all supported |
| Layout | `BASE_W = 480`, `BASE_H = 720`, `resize()` | Canvas is scaled to fit the viewport |
| Game state | `resetGame()`, `update(dt)`, `draw()` | `update` handles physics, spawning, and collisions; `draw` renders with canvas 2D |
| Weapons | `fireWeapon()`, `applyPowerUp()`, `maybeSpawnPowerUp()` | Weapon power-ups raise weapon level |
| Bombs | `useBomb()`, `flashBomb()` | Screen-clearing smart bomb |
| Bosses | `spawnBoss()`, `BOSS_PATTERNS`, `updateBoss()`, `killBoss()`, `fire*` pattern helpers | Each boss has a list of attack patterns; some are seeded |
| Lifecycle | `loseLife()`, `endGame()`, `togglePause()` | Game over shows the overlay with stats |
| Loop | `loop(now)` | `requestAnimationFrame` loop; `dt` is clamped to 0.05 s |
| Accessibility | `announce()`, `srLive` region, `reducedMotion` toggle | Screen shake, flashes, and pulses can be disabled |

## Controls (for reference when testing)

- **Desktop:** mouse to steer and click to fire; arrow keys or WASD to move; Space to fire; X or Shift for bomb; P or Esc to pause.
- **Touch:** drag on the left zone to fly with the virtual joystick; tap FIRE; tap BOMB.

## Conventions

- **Keep it a single file** unless the user asks to split it. Don't add a build system, framework, or dependencies without a request.
- **Plain JavaScript and canvas 2D.** Match the existing style: `const`/`let`, arrow functions for callbacks, 2-space indentation, single quotes.
- **Tune gameplay through the config objects** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `KILLS_PER_STAGE`) rather than hardcoding numbers in update logic.
- **Frame-rate independence:** movement and timers should use `dt` (seconds), not per-frame counts.
- **Respect the reduced-motion setting** for any new screen shake, flash, or pulse effect; route it through the existing `screenShake` / `flashHit` / `flashBomb` helpers.
- **Persisted state** uses `localStorage` with the `skyfighter.` prefix. Wrap new storage access in a way that survives blocked storage if you add any.
- **Announce important state changes** to screen readers through `announce()`.

## Working on the game

- No automated tests or linters exist. Verify changes by loading `index.html` in a browser and playing through the affected path (start, fire, bomb, stage change, boss fight, pause, game over).
- Check both desktop and touch layouts when touching the HUD, overlay, or touch controls. The layout must work at phone width.
- Check that the game still runs in the three difficulty modes if you change shared config.

## Git

- Development branch for this work: `claude/gifted-tesla-p4mfgh`.
- Commit with clear messages and push with `git push -u origin <branch>`.
- Don't open a pull request unless the user asks for one.
