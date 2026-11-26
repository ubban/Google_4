# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up written as a single self-contained HTML5 canvas page.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript (~1,670 lines, inline `<style>` and one inline `<script>`). |
| `README.md` | Placeholder title only. |

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. The game runs by opening `index.html` in a browser.

## Running and checking changes

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` and visit `/index.html`).
- **Lint/build/test:** none configured. Verify changes by loading the page and playing; for JavaScript syntax, `node --check` can be run on an extracted script block.
- Keep it working as a single file. Do not split it into modules or add a toolchain unless the user asks.

## Code layout inside `index.html`

The inline script is one IIFE with `'use strict'`. Major sections, in source order:

1. **DOM references** and `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`) used with `localStorage`.
2. **Adaptive quality** (`trackPerf`): samples frame times and steps a quality level 0–2.
3. **Config tables**
   - `DIFFICULTIES` (easy / normal / hard): spawn rate, enemy speed and fire chance, lives, bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled).
4. **Audio** (`ensureAudio`, `beep`, `noiseBurst`): Web Audio synthesis only; no audio files. Respects the mute setting.
5. **HUD and effects**: hearts, bombs, combo pop, stage banner, screen shake, hit/bomb flashes.
6. **Layout and input**: `resize` (fixed `BASE_W = 480`, `BASE_H = 720` logical canvas, scaled to the window), keyboard, mouse, touch joystick, and fire/bomb buttons.
7. **Game logic**: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `applyPowerUp`.
8. **Boss patterns**: `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring), driven by `updateBoss`.
9. **Lifecycle**: `loseLife`, `killBoss`, `endGame`, `togglePause`.
10. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
11. **Main loop**: `loop(now)` caps `dt` at 0.05 s and calls `update` then `draw` only while running and not paused.

Game state is plain module-level `let`/`const` variables inside the IIFE. There are no classes and no framework.

## Conventions

- **Vanilla JS only.** No frameworks, no npm packages, no CDN scripts unless the user asks for them.
- **Keep the single-file structure.** New CSS goes in the existing `<style>` block, new JS in the existing `<script>` block.
- **Logical coordinates.** Gameplay is authored in the 480×720 logical space; rendering and input are scaled from that. Use logical units when adding entities.
- **Respect accessibility settings.** `reducedMotion` (from the OS preference or the saved toggle) dampens screen shake (`screenShake` scales magnitude and duration down) and should be honored by any new motion or flash effect; the live region `#srLive` is used for announcements via `announce()`.
- **Persist only through the `skyfighter.*` keys** listed above, and wrap storage access so the game still works when `localStorage` is unavailable.
- **Support both desktop and touch.** Any new control needs a keyboard/mouse path and a touch path (`touchControls` is shown on touch devices).
- Use the existing code style: 2-space indentation, single quotes, `const` by default, short section comments with `// ---------- Name ----------`.

## Controls (for reference when editing input)

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire (holding fires continuously).
- X or Shift: bomb (clears the screen, damages bosses).
- P or Esc: pause.
- Touch: drag in the left zone to fly, FIRE and BOMB buttons on the right.

## Game rules worth knowing before changing balance

- Power-ups: `life`, `bomb`, `weapon` (Vulcan/Laser with 3 power levels).
- Score is multiplied by a kill-chain combo; the chain breaks on a hit.
- A stage advances after `KILLS_PER_STAGE` kills and ends with a boss fight that shows a health bar.
- Best score is stored locally per browser; there is no server.

## Git and workflow

- Default working branch for this session: `claude/gifted-tesla-yhwd7v`. Commit and push there.
- Do not open pull requests unless the user explicitly asks.
- Keep commits focused. Describe gameplay changes in terms of player-visible behavior.
