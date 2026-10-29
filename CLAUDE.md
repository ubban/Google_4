# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. There is no build step, no package manager, and no test suite.

- `index.html` (~1,670 lines): all markup, CSS (`<style>` block, lines 10–360), and JavaScript (`<script>` block, starting around line 422). The game logic lives in one IIFE.
- `README.md`: a stub. It has only the repo name and a timestamp.

To play, open `index.html` in a browser. Keep it that way: do not add a bundler, framework, or external dependency unless the user asks.

## Working on the code

- Edit `index.html` directly. Keep changes small and local, and match the surrounding style: 2-space indent, single quotes in JS, `const` by default, `camelCase` functions, `SCREAMING_SNAKE_CASE` for config tables.
- Run the game in a browser to verify changes. There are no automated tests or linters configured. For a quick syntax check of the script block, you can extract it and run `node --check`.
- Use `'use strict'` semantics as the file already does. The script runs inside an IIFE, so nothing is exposed on `window`.

## Architecture

The game loop and state are plain module-level variables inside the IIFE.

- **Loop:** `loop(now)` runs on `requestAnimationFrame`. `dt` is clamped to 50 ms. While `running && !paused`, it calls `update(dt)` then `draw()`. `trackPerf(dt)` runs every frame and steps a `quality` level (0 full, 1 medium, 2 low) based on the rolling average frame time.
- **Update / draw:** `update(dt)` (around line 1148) handles movement, spawning, collisions, and scoring. `draw()` (around line 1520) renders everything to the 2D canvas. Helpers such as `drawPlane`, `drawBossShip`, and `drawClouds` handle sprites.
- **Config tables:** `DIFFICULTIES` (spawn rate, enemy speed and fire chance, lives, bombs, boss HP multiplier), `THEMES` (six stage palettes, cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six bosses, with `BOSS_PATTERNS` defining their attack patterns). Tune balance here before touching logic.
- **Stages:** `KILLS_PER_STAGE` (18) controls stage progression. A stage banner is shown on advance.
- **Weapons and power-ups:** `fireWeapon()`, `applyPowerUp()`, and `maybeSpawnPowerUp()`. Bombs are handled by `useBomb()`.
- **Scoring:** `addScore()` applies a combo multiplier. `breakChain()` resets it.
- **Canvas sizing:** the logical play area is `BASE_W` × `BASE_H` (480 × 720). `resize()` scales it to the window.
- **Audio:** Web Audio via a lazily created `audioCtx` (`ensureAudio()`). Sound effects are synthesized with `beep()` and `noiseBurst()`. Nothing is loaded from disk. Audio starts only after a user gesture.

## Input

- **Keyboard:** Arrow keys or WASD move. `Space` fires. `X`, `Shift` drop a bomb. `P` or `Escape` pauses.
- **Touch:** a virtual joystick (`moveZone`), plus fire and bomb buttons.
- **Mouse / pointer:** `canvasPointFromEvent()` maps pointer coordinates into the logical canvas space.

Input listeners live on `window` and on the touch control elements. Keep them consistent with the existing `keys[e.code]` pattern.

## Persistence

The game stores four keys in `localStorage`. Read and write them only through the existing constants:

| Constant | Key | Purpose |
|---|---|---|
| `STORAGE_BEST` | `skyfighter.bestScore` | High score |
| `STORAGE_MUTE` | `skyfighter.muted` | `'1'` when muted |
| `STORAGE_DIFF` | `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `STORAGE_REDUCED` | `skyfighter.reducedMotion` | `'1'` / `'0'`, overrides the OS preference |

Reduced motion defaults to the `prefers-reduced-motion` media query until the user sets it explicitly. When it is on, `screenShake()` caps shake duration at 0.1 s, the hit/bomb flash and some HUD animations are suppressed via `body.reduced-motion` CSS rules, and the flash helper returns early.

Accessibility: `announce()` writes to the `srLive` live region for screen readers. Keep that working when you add new status messages.

## Conventions and gotchas

- Do not add dependencies, a build step, or files outside `index.html` unless the user asks. A single self-contained file is the design.
- `localStorage` access can throw in some privacy modes. The current code does not guard it. If you touch those lines, consider a try/catch.
- Do not change the `dt` clamp or the frame-time quality logic without measuring. They keep the game playable on slow devices.
- Keep HUD changes in sync across the DOM elements (`scoreEl`, `heartsEl`, `bombsEl`, and so on) and the `draw()` function.
- Touch and keyboard controls must both work for any new player action.

## Git

- Development branch for this session: `claude/gifted-tesla-dckm13`. Commit and push there. Do not push to `main` unless asked.
- Write clear, descriptive commit messages in the imperative mood.
