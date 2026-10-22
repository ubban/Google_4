# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas app.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–359), and JavaScript (`<script>`, ~lines 422–1667).
- `README.md` — minimal project title only.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Do not add one unless asked.

## Running locally

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. No install step is needed.

## Code layout inside `index.html`

The script is one IIFE (`'use strict'`) with these sections, roughly in order:

1. **DOM references and storage keys** — `getElementById` lookups; `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
2. **Tuning data** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky/cloud/sun palettes per stage), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here.
3. **Audio** — Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`, `panFor`). Audio is created lazily on first user gesture.
4. **Adaptive quality** — `trackPerf` lowers/raises a `quality` level (0–2) based on a rolling frame-time average.
5. **Input** — keyboard (`keydown`/`keyup` on `e.code`), touch joystick and fire/bomb buttons, and mouse/pointer movement on the canvas.
6. **Game state and entities** — `resetGame`, `update(dt)` (main simulation), `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `breakChain`/`addScore` (kill-chain multiplier).
7. **Boss patterns** — `BOSS_PATTERNS` (bullet-pattern functions such as radial, spiral, cross, homing) driven by `updateBoss`; `killBoss`.
8. **Lifecycle** — `loseLife`, `endGame`, `togglePause`.
9. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`); drawing is canvas-only, sized by `resize()` against `BASE_W` × `BASE_H` (480 × 720).
10. **Main loop** — `loop(now)` uses `requestAnimationFrame`.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Don't introduce frameworks or external CDN dependencies.
- **Match the existing style.** Two-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and short section comments in the `// ---------- Name ----------` form.
- **Keep the game's coordinate space.** Gameplay uses the fixed `BASE_W`/`BASE_H` space; don't hard-code canvas pixel sizes.
- **Persist via the existing `skyfighter.*` keys.** Wrap new `localStorage` access in the same pattern used now, and use a new key with the `skyfighter.` prefix.
- **Accessibility and mobile.** Respect the `reduced-motion` setting (`reducedMotion` flag and `body.reduced-motion` class), announce important state changes through `announce()` (the `srLive` region), and keep touch controls working (the viewport disables user scaling).
- **Preserve `preventDefault` on Space** and the existing key bindings unless the change explicitly alters controls.

## Controls (for reference when editing input)

| Action | Keyboard | Mouse (canvas) | Touch |
| --- | --- | --- | --- |
| Move | Arrow keys / WASD (via `keys[e.code]`) | Move pointer to steer (`mouseActive`) | Drag in left joystick zone |
| Fire | Space (held) | Press and hold (`firing`) | Fire button (hold) |
| Bomb | X or Shift | — | Bomb button |
| Pause | P or Escape | — | On-screen pause button (also clickable with a mouse) |

## Verifying changes

Since there is no automated test harness:

1. Open `index.html` in a browser (or serve it) and confirm there are no console errors.
2. Play through at least one stage to exercise the affected code path, and check a boss fight if you touched boss logic, power-ups, or bombs.
3. For layout changes, check both a desktop viewport and a phone-sized viewport.

Say clearly in your summary when you could not run the game in a browser.

## Git

- Develop on the branch the session specifies; don't push to other branches without permission.
- Write descriptive commit messages in the imperative mood, matching the existing history (e.g. "Add Sky Fighter: …").
