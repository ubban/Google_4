# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- `index.html` contains all markup, CSS (`<style>`, ~lines 10–361), and game logic (one inline `<script>` IIFE, ~lines 422–1668).
- No build step, no package manager, no dependencies. Open the file in a browser or serve it statically to run it.
- `README.md` is a placeholder and contains no setup instructions.

## Running and verifying

- Run: open `index.html` directly, or `python3 -m http.server` from the repo root and visit `http://localhost:8000`.
- There is no test suite or linter configured. Verify changes by loading the game in a browser and playing through the affected path (start, fire, weapon pickup, bomb, boss, pause, game over).
- Check the browser console for errors after any change to the script.

## Code layout inside `index.html`

The script is organized by `// ---------- Section ----------` comments. Key areas, in order:

| Area | Purpose |
| --- | --- |
| DOM refs and `STORAGE_*` keys | Element lookups; `localStorage` keys prefixed `skyfighter.` |
| Adaptive performance | `trackPerf` drops render `quality` (0–2) when frame time is high |
| Tuning tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, the `sfx` table; audio is created lazily on first user gesture |
| HUD and effects | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake` |
| Layout | `BASE_W = 480`, `BASE_H = 720` logical canvas size; `resize()` scales to the viewport |
| Input | Keyboard (`keys`), mouse, touch joystick, fire and bomb buttons |
| Game state | `resetGame()` builds `player`, `enemies`, `bullets`, `powerUps`, etc. |
| Spawning and combat | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `addScore`, combo chain |
| Boss patterns | `BOSS_PATTERNS` (radial, aimed spread, spiral, cross, homing, double ring) driven by `updateBoss` |
| Flow | `loseLife`, `killBoss`, `endGame`, `togglePause` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Main loop | `update(dt)` each frame, then `draw()`; `loop(now)` clamps `dt` to 50 ms |

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The file is intentionally self-contained.
- Wrap the script in the existing `(function () { 'use strict'; ... })();` IIFE. Do not add globals.
- Match the surrounding style: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and the existing `// ---------- Section ----------` banners for new sections.
- Put new tuning values in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Time-based motion multiplies by `dt` (seconds). Do not tie movement to frame count.
- Persist settings only through the `skyfighter.*` `localStorage` keys, and wrap new storage reads in a fallback so the game still runs when storage is unavailable.
- Accessibility features exist and should be preserved: `announce()` writes to the `aria-live` region, and `reducedMotion` weakens screen shake and skips flashes. New visual effects that flash or shake should respect `reducedMotion`.
- Controls must keep working on both desktop (keyboard, mouse) and touch devices. The on-screen help text (`#controlsHint`) lists the controls; update it if bindings change.

## Controls (for reference when changing input code)

- Move: mouse, Arrow keys, or WASD. Touch: drag left side to fly.
- Fire: Space (or hold mouse / tap FIRE).
- Bomb: `X` or Shift (or tap BOMB).
- Pause: `P` or Esc.
- Mute, motion toggle, and difficulty are set from the overlay and HUD buttons.

## Git workflow

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-v3q8oz`). Do not push to other branches without explicit permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
