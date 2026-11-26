# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single HTML5 canvas page.

- `index.html` — the entire game: inline CSS (`<style>`), markup, and one inline IIFE `<script>` (starts around line 422). There is no build step, no bundler, and no external dependencies.
- `README.md` — minimal project readme (title only).

There are no tests, no package manifest, and no CI configuration in the repo.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code structure of `index.html`

- **Config tables** (top of the script): `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses, selected by `(stage - 1) % BOSSES.length`).
- **Persistence** uses `localStorage` with keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should stay tolerant of storage being unavailable.
- **Audio** is generated with the Web Audio API (`beep`, `noiseBurst`, and related helpers); there are no audio files.
- **Rendering** is a single `draw()` pass on a fixed logical canvas of `BASE_W × BASE_H` = 480 × 720, scaled by `resize()`.
- **Simulation** is `update(dt)`, driven by `loop(now)`. Weapons (`fireWeapon`), power-ups (`applyPowerUp`), bombs (`useBomb`), boss attack patterns (`BOSS_PATTERNS`, `updateBoss`), and the kill-chain score multiplier are all here.
- **Input**: keyboard (`keys[e.code]`), canvas pointer/touch input, an on-screen virtual joystick, and an on-screen fire button for mobile.

## Controls (as implemented)

| Action | Keys |
| --- | --- |
| Move | Arrow keys or WASD |
| Fire | Space, or hold the on-screen fire button (mouse/touch) |
| Smart bomb | X or Shift |
| Pause | P or Escape |

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a restructure. Do not add a bundler or framework without an explicit request.
- Put tunable numbers in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Preserve the mobile layout: `touch-action: none`, `viewport-fit=cover`, safe-area insets, and the joystick must keep working at phone widths.
- Respect `reducedMotion` for any new screen shake, flashing, or animation.
- Use the `Sky Fighter` naming and the `skyfighter.` storage-key prefix for anything new.
- Do not add external CDN scripts or fonts; the game currently loads nothing over the network.

## Verifying changes

- Since there are no automated tests, verify changes by loading the page in a browser and exercising the affected path (menu → play → weapon/power-up/bomb → boss → game over).
- Check the console for errors after each change.
- For layout changes, check both a desktop viewport and a phone-sized viewport.

## Git

- Development happens on the branch designated for the session; do not push to other branches without permission.
- Do not open pull requests unless explicitly asked.
