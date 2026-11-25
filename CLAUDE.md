# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up written as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–361), and JavaScript (`<script>`, ~lines 422–1667). No external assets or libraries.
- `README.md` — a one-line placeholder.

There is no build step, package manager, bundler, linter, or test suite. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify gameplay changes by playing in a browser and checking both desktop (keyboard/mouse) and touch controls.

## Code structure of `index.html`

The game script is one IIFE (`'use strict'`) with these sections, in order:

1. **DOM lookups** — `document.getElementById` references for the canvas, HUD, overlay, and buttons.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`.
3. **Config tables** — `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
4. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is created lazily on the first user gesture.
5. **HUD and effects** — `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`.
6. **Input** — keyboard (`keys` map), touch joystick and fire/bomb buttons, and mouse steering on the canvas.
7. **Game state** — `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore`, `breakChain`, `useBomb`, `fireWeapon`.
8. **Update** — `update(dt)` advances the simulation; `updateBoss(dt)`, `applyPowerUp`, `loseLife`, `endGame`, `killBoss`.
9. **Boss patterns** — `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring).
10. **Render** — `draw()` and the `draw*` helpers. Rendering is canvas-only, drawn at a fixed `BASE_W × BASE_H` (480 × 720) and scaled by `resize()`.
11. **Main loop** — `loop(now)` caps `dt` at 0.05 s, updates, then draws. `trackPerf` adapts a `quality` level (0–2) from frame times.

Keep these conventions when editing:

- **Game logic and rendering stay separate.** `update` mutates state; `draw` only reads it.
- **Frame-rate independence.** Movement and timers use `dt` (seconds). Do not hard-code per-frame increments for anything that should be time-based.
- **Tune through the config tables.** Difficulty numbers, enemy stats, boss HP multipliers, and theme colors belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES`, not inline literals.
- **Respect reduced motion.** Screen shake, hit flashes, and the low-health pulse must be gated on `reducedMotion` (toggled by `#motionBtn`).
- **Keep the accessibility hooks.** Announcements go through `announce()` into the `#srLive` live region.

## Controls (for manual testing)

- **Desktop:** mouse to steer and click to fire; arrows or WASD to move; Space to fire; X or Shift for a bomb; P or Esc to pause.
- **Mobile:** drag on the left zone to fly, tap FIRE, tap BOMB.

## Persistence

Only `localStorage` is used, under the `skyfighter.*` keys listed above. Wrap any new storage access in a way that still works when storage is unavailable, such as private browsing. Do not add server-side persistence without discussing it first.

## Git workflow

- Develop on the branch assigned for the session. Do not push to `main` without explicit permission.
- Commit messages should describe the gameplay or code change and why. Keep the subject line short.
- Do not open pull requests unless asked.

## Conventions for new files

- Keep the project dependency-free unless there is a clear reason to add tooling. Anything added to the project should be documented here.
- If the game grows beyond one file, split CSS and JS into their own files and update the structure section above.
