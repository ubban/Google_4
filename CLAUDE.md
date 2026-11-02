# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. Everything lives in one file:

- `index.html` (~1670 lines) — inline `<style>`, markup, and one IIFE-wrapped `<script>` containing all game logic and rendering.
- `README.md` — minimal placeholder.

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. The game runs by opening `index.html` in a browser.

## Running and testing

- Run: open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing in a browser, checking both desktop (mouse + keyboard) and mobile-width (touch) layouts.
- Check the browser console for errors after any change.

## Code layout inside `index.html`

The script is one IIFE. Roughly, in order:

1. **Constants and config** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`-related data.
2. **Persistence keys** — `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`.
3. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and a per-sound table (`EXPLODE_SFX` etc.). Audio is created lazily on first user gesture.
4. **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
5. **Input** — `keys` state map (keyed by `KeyboardEvent.code`), pointer/mouse steering, a touch joystick (`joystickStart/Move/End`), and `canvasPointFromEvent`.
6. **Game state** — `resetGame()` is the single place to reinitialize run state.
7. **Drawing** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
8. **Simulation** — `update(dt)` drives movement, collisions (`rectsOverlap`), spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`), bombs (`useBomb`), scoring (`addScore`, `breakChain`), power-ups (`applyPowerUp`), and boss attacks (`fire*` pattern helpers, `updateBoss`, `killBoss`).
9. **Lifecycle** — `loseLife`, `endGame`, `togglePause`.
10. **Main loop** — `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` + `draw` when running and not paused.

Canvas logical size is `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()` to fit the viewport.

## Controls (keep these in sync with the on-screen hint)

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire.
- X or Shift: bomb.
- P or Esc: pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

The control hint text is in `#controlsHint` in the markup. If you change a binding, update that string too.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Don't add a build system or dependencies without being asked.
- **Vanilla JS and Canvas 2D only.** No frameworks, no libraries.
- **Tune through the constants.** Balance changes (enemy speed, spawn rate, lives, boss HP) belong in `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES`, not hard-coded in update logic.
- **Respect reduced motion.** Visual effects that shake or flash should check the `reducedMotion` setting.
- **Keep the frame loop cheap.** `update` and `draw` run every frame; avoid per-frame allocations in hot paths where practical.
- **Persist only through the `skyfighter.*` keys.** Wrap new `localStorage` access in the same pattern the file already uses.
- **Mobile must keep working.** Preserve `touch-action: none`, the safe-area insets, and the responsive HUD wrap when changing layout.

## Git workflow

- Default branch: `main`. Feature work for this session goes on `claude/gifted-tesla-f8usop`.
- Commit messages should describe the gameplay or code change, not the file edited.
- Don't open pull requests unless explicitly asked.
