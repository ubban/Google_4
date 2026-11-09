# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (in `<style>`), and JavaScript (one IIFE in `<script>`). No build step, no bundler, no dependencies.
- `README.md` — minimal project readme.

There is no test suite, package manifest, or linter configured. Verify changes by opening `index.html` in a browser (or with the `run` skill / Playwright) and playing through the affected behavior.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Code layout (inside `index.html`)

The script is organized into commented sections. Keep new code in the matching section:

- **DOM references** — `const xEl = document.getElementById(...)` block at the top of the IIFE.
- **Storage keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Any new persisted setting should follow this `skyfighter.*` naming.
- **Adaptive performance** — `trackPerf` lowers/raises a `quality` level (0–2) based on rolling frame time.
- **Config tables** — `DIFFICULTIES`, `THEMES` (stage background palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, and `KILLS_PER_STAGE`. Balance changes usually belong here rather than in logic.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` map. Audio is created lazily on the first user gesture; respect the mute flag.
- **HUD and feedback** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
- **Input** — keyboard listeners on `window`, a virtual joystick and fire/bomb buttons for touch, and `canvasPointFromEvent` for pointer coordinates.
- **Game state** — `resetGame`, `update(dt)`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `endGame`, `togglePause`.
- **Bosses** — `spawnBoss`, `updateBoss`, `killBoss`, and the pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **Rendering** — `draw`, `drawPlane`, `drawBossShip`, `drawClouds`. Rendering reads game state; it should not mutate it.
- **Main loop** — `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05 s, and calls `update` then `draw` only while running and not paused.

## Conventions

- **Single file.** Keep everything in `index.html` unless the user asks for a split. Do not introduce a build system, framework, or external dependencies without an explicit request.
- **Vanilla JS, ES2015+ style.** Use `const`/`let`, arrow functions, and template literals consistent with the existing code. Keep the `'use strict'` IIFE wrapper; do not add globals.
- **Canvas coordinates.** Game logic uses the fixed base resolution `BASE_W × BASE_H` (480 × 720); `resize()` scales the canvas to the viewport. Write positions and sizes in base units.
- **Frame-rate independence.** Movement and timers multiply by `dt` (seconds). Do not hard-code per-frame increments.
- **Persistence.** `localStorage` access is currently unguarded at load time. If you add new storage reads/writes, wrap them so the game still runs when storage is unavailable (private mode, blocked site data).
- **Accessibility.** Respect `reducedMotion` for screen shake and other motion effects. Announce important state changes through `announce()`.
- **Mobile.** The layout must work on touch devices with safe-area insets (`env(safe-area-inset-*)`), and touch handlers use `{ passive: false }` where they call `preventDefault`. Keep the no-scroll viewport behavior.
- **Controls** (keep documentation and HUD in sync if these change):
  - Move: arrow keys / WASD or the on-screen joystick
  - Fire: Space (hold) or the fire button
  - Bomb: `X` or Shift, or the bomb button
  - Pause: `P` or Escape
  - Mute toggle and motion toggle are in the HUD

## Game design notes

- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Stages** advance every `KILLS_PER_STAGE` (18) kills, cycle through `THEMES`, and end in a boss from `BOSSES`. Each boss has a matching entry in `BOSS_PATTERNS`.
- **Enemies** are `grunt`, `interceptor` (fast, weaving), and `tank` (4 HP, unlocked at stage 2).
- **Weapons** have a Vulcan/Laser system with three power levels; power-ups include weapon level, bomb, and extra life.
- **Score** uses a kill-chain multiplier (`addScore`, `breakChain`).

## Workflow for changes

1. Read the relevant section of `index.html` before editing; the file is large, so search by function name.
2. Make the smallest change that satisfies the request. Do not reformat unrelated code.
3. Test in a browser: start a game, exercise the changed path on both keyboard and touch layouts when relevant, and check the console for errors.
4. Commit with a clear message describing the gameplay or code change.
