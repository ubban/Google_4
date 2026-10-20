# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-file HTML5 canvas game. There is no build step, no framework, and no dependencies.

- `index.html` — the entire game: inline `<style>`, markup, and one IIFE-wrapped `<script>` (~1,670 lines).
- `README.md` — placeholder; contains no real project documentation.

To play, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

```
index.html   # Full game: CSS (lines ~10–359), DOM (~361–421), game script (~422–1667)
README.md
```

Inside the game script, the code is grouped by section comments (`// ---------- Name ----------`). Use them to navigate:

| Area | What it holds |
|---|---|
| Adaptive performance | `trackPerf`, `quality` (0 full / 1 medium / 2 low) |
| Config tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Persistence | `STORAGE_*` keys in `localStorage` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx.*`, `EXPLODE_SFX` |
| HUD & feedback | `renderHearts`, `renderBombs`, combo pop, stage banner, screen shake, flash |
| Input | keyboard `keys` map, pointer/touch joystick, fire and bomb buttons |
| Entities & spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion` |
| Combat | `fireWeapon`, `useBomb`, `applyPowerUp`, `BOSS_PATTERNS` and `fire*` helpers |
| Main loop | `update(dt)`, `updateBoss`, `draw()`, `loop(now)` |
| Game state | `resetGame`, `loseLife`, `killBoss`, `endGame`, `togglePause` |

## Controls (for reference when testing)

- **Move:** arrow keys or WASD (the `keys` map). Mouse hover steers the plane toward the pointer while it is over the canvas; keyboard and joystick input take priority when active.
- **Fire:** hold Space, hold the mouse button on the canvas, or hold the on-screen fire button.
- **Bomb:** `X` or either Shift key, or the on-screen bomb button.
- **Pause:** `P` or `Escape`.
- Touch devices get a joystick (`moveZone`) plus on-screen fire and bomb buttons.

## Conventions

- **Single file.** Keep the game in `index.html`. Do not split it into modules or add a bundler unless the user asks for that change.
- **No dependencies.** Use plain JavaScript and the Canvas 2D API. Do not add npm packages, CDN scripts, or frameworks without an explicit request.
- **Style matches the file.** Use `const`/`let`, arrow functions for callbacks, two-space indentation, single quotes in JS, and section divider comments of the form `// ---------- Name ----------`. Add a short comment only where the logic is not obvious.
- **Keep the IIFE.** All game code lives inside `(function () { 'use strict'; ... })();`. Do not leak new globals; use `window` only for the existing listeners.
- **Tune with data.** Balance changes (enemy stats, difficulty, boss HP multipliers, stage length) belong in the config tables near the top of the script rather than scattered in logic.
- **Persistence keys** are namespaced as `skyfighter.*`. Any new saved setting should follow that pattern and be read with a fallback default.
- **Accessibility.** Respect `prefers-reduced-motion` (the `reducedMotion` flag and `reduced-motion` body class), and announce important state changes through the `srLive` live region.
- **Performance.** The loop is driven by `requestAnimationFrame` and uses `dt`. Avoid per-frame allocations in hot paths, and respect the adaptive `quality` level when adding visual effects.

## Testing and verification

There is no test suite, linter, or build configured in this repo. Verify changes by:

1. Opening `index.html` in a browser (or serving it locally) and playing through at least one full stage.
2. Checking all three difficulties, a boss fight, bomb use, pause/resume, and game over.
3. Confirming that the best score, mute, difficulty, and reduced-motion settings persist across reloads.
4. Checking the layout on a narrow (phone-width) viewport and touch controls, since the canvas resizes via `resize()`.

If you add tooling later (for example, a test runner or linter), document the commands here.

## Git workflow

- Develop on the branch named by the session or task. Do not push to other branches without permission.
- Write commit messages in the imperative mood with a short summary line, followed by a body for non-trivial changes. The existing history follows this style (see `git log`).
- Do not open pull requests unless the user asks for one.
