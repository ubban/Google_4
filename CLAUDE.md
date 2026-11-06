# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- Entry point and entire app: `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>` IIFE).
- `README.md` is a placeholder and contains no setup instructions.
- There is no build step, package manager, bundler, test suite, or linter. Nothing needs to be installed.

## Running locally

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code layout (inside `index.html`)

The game script is one `(function () { 'use strict'; ... })();` block. Major sections, in source order:

- **DOM refs and constants**: element lookups, `STORAGE_*` keys, the `DIFFICULTIES`, `THEMES` (stage backdrops), `ENEMY_TYPES`, and `BOSSES` tables. `KILLS_PER_STAGE` controls stage progression.
- **Adaptive performance**: `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) that the renderer uses to reduce effects.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is created lazily on the first user gesture and can be muted.
- **Input**: keyboard (`keys` map), mouse (`canvasPointFromEvent`), and touch (virtual joystick, FIRE and BOMB buttons).
- **Game state**: `resetGame` initializes state. `update(dt)` runs game logic, and `draw()` renders. `loop(now)` drives both through `requestAnimationFrame`, with `dt` capped at 0.05 s.
- **Weapons and power-ups**: `fireWeapon` (Vulcan/Laser, 3 power levels), `applyPowerUp`, `maybeSpawnPowerUp`, `useBomb`.
- **Bosses**: `spawnBoss`, `updateBoss`, `killBoss`, and the `BOSS_PATTERNS` array of bullet patterns (radial, spread, spiral, cross, homing, and so on).
- **Lifecycle**: `loseLife`, `endGame`, `togglePause`.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds` (parallax), `draw`.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

| Key | Holds |
| --- | --- |
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; defaults to the `prefers-reduced-motion` media query when unset |

Keep these keys stable, because changing them resets players' saved state. Storage calls are not wrapped in `try/catch`, so a blocked-storage environment can throw. Wrap them if you touch that code.

## Controls (for reference when testing)

- Desktop: mouse to steer and click to fire, or arrows/WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Touch: drag on the left to fly, tap FIRE, tap BOMB.

## Conventions

- Keep the game in the single `index.html` file unless the user asks for a split. Do not add a build step or dependencies without a clear reason.
- Put new tuning values in the existing constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and so on) rather than hard-coding magic numbers in logic.
- Gameplay code is written in the same style as the surrounding code: `const`/`let`, arrow functions in callbacks, and two-space indentation.
- Respect `reducedMotion` and `quality` when adding visual effects, such as screen shake, particles, or glow.
- Keep the screen-reader hook working: `announce()` writes into the `srLive` live region (`role="status"`, `aria-live="polite"`).

## Verification

There are no automated tests. After changes, open the game in a browser and check at least:

1. Start, play, lose all lives, and reach the game-over screen with the best score updated.
2. Pause and resume with P and Esc.
3. Difficulty buttons and mute persist across a reload.
4. A stage boss appears and its health bar updates.
5. The layout still works at phone width, using the touch controls.

For scripted checks, a headless Chromium with Playwright can load the file and drive the same flow.

## Git

- Development branch for this task: `claude/gifted-tesla-0b0x70`. Push there with `git push -u origin <branch>`.
- Commit messages describe the change in the imperative mood and explain why when that is not obvious.
