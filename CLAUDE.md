# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up game, delivered as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), canvas element, and all JavaScript (one IIFE in a `<script>` block, ~lines 422–1668).
- `README.md` — minimal project readme (currently just a title).

There is no build system, package manager, bundler, test suite, or linter configured. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

Verify changes by loading the page and playing: start a run, fire, use a bomb, pause/resume, lose a life, and check the game-over screen. Also check the layout at phone width, since touch controls are part of the game.

## Architecture of `index.html`

The script is a single `(function () { 'use strict'; ... })();` closure. Everything is private to it; there are no modules and no globals.

Key sections, in roughly the order they appear in the script:

1. **DOM references** — `const xEl = document.getElementById(...)` for every HUD and overlay element. IDs in markup must match these lookups.
2. **Constants** — `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`), `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, starting lives/bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
3. **Audio** — `ensureAudio()` lazily creates the `AudioContext` (must run from a user gesture), plus `beep`, `noiseBurst`, and the `sfx` object. Sounds are synthesized, not loaded from files.
4. **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
5. **Input** — keyboard (`keys[e.code]`), touch joystick and fire/bomb buttons, and mouse/pointer on the canvas. Canvas coordinates come from `canvasPointFromEvent`.
6. **Game state & flow** — `resetGame`, `update(dt)`, `loseLife`, `endGame`, `togglePause`. Persistence of best score happens in `endGame`.
7. **Spawning & entities** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, collision via `rectsOverlap`.
8. **Weapons & bosses** — `fireWeapon` (Vulcan/Laser, levels 1–3), `applyPowerUp`, bomb logic in `useBomb`, boss attack patterns in `BOSS_PATTERNS` (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `updateBoss`, `killBoss`.
9. **Rendering** — `draw()` and helpers `drawPlane`, `drawBossShip`, `drawClouds`. Parallax clouds and sun glow are drawn every frame.
10. **Main loop** — `loop(now)` drives `update` and `draw` via `requestAnimationFrame`, with a delta-time clamp. `trackPerf` samples frame timing.

The canvas is a fixed logical size (`BASE_W = 480`, `BASE_H = 720`) scaled by `resize()` to fit the viewport.

### Controls

- Move: arrow keys or WASD, touch joystick, or mouse on canvas.
- Fire: Space (held), touch fire button, or mouse button.
- Bomb: `X` or Shift, or touch bomb button.
- Pause: `P` or `Escape`.

## Conventions

- **Keep it one file.** The game is intentionally a single HTML file with no dependencies. Do not introduce a build step, npm packages, or split files unless the user asks for it.
- **Stay inside the IIFE.** New code should live inside the existing closure. Don't add top-level globals.
- **Use the existing structure.** Put new constants with the other constants, new HUD updates near the other HUD helpers, and new enemy or boss behavior alongside the related `spawn*`, `update*`, and `fire*` functions.
- **Time-based updates.** Movement and timers use the `dt` passed to `update(dt)`. Don't tie behavior to frame count.
- **Respect reduced motion.** When adding visual effects that shake, flash, or pulse, add a matching `body.reduced-motion` CSS rule so the existing toggle disables them.
- **Keep audio user-gated.** Audio must start only from a user gesture through `ensureAudio()`. Don't create an `AudioContext` at load time.
- **Persistence.** Use the `STORAGE_*` keys for anything saved. Keep the `skyfighter.` prefix.
- **Accessibility.** Announce important state changes through the existing `announce()` helper, which feeds the `srLive` live region. Keep overlay buttons and controls keyboard-reachable.
- **Mobile.** Changes to the HUD or controls must work at phone width and with touch input (`touchControls`, `moveZone`, `fireBtn`, `bombBtn`).

## Known gaps

- `localStorage` reads and writes are not wrapped in `try/catch`. In browsers with storage blocked (for example, some private modes or strict settings), these calls can throw. If you touch persistence code, consider guarding it.
- There are no automated tests. Verification is manual, in a browser.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-1vmrez`). Don't push to other branches without explicit permission.
- Write clear, descriptive commit messages that explain the change, not just the file touched.
- Don't open a pull request unless the user asks for one.
