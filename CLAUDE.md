# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-file browser game, **Sky Fighter** — a Raiden-style vertical shoot-em-up. There is no build step, no package manager, and no test suite.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines). |
| `README.md` | Placeholder; contains only the repo title and a stray timestamp. |

## Running and verifying

- No install, build, lint, or test commands exist. Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify changes by playing in a browser. Check at least: start/restart, each difficulty, a stage transition, a boss fight, bomb use, pause/resume, and game over with a best-score update.
- Also check touch controls when editing input code, since they are only enabled on touch devices (`isTouchDevice`).

## Architecture of `index.html`

Everything runs inside a single IIFE (`(function () { 'use strict'; ... })();`). Nothing is exposed on `window`, so keep new code inside that scope.

Major sections, in file order (search for the `// ----------` comment banners):

1. **DOM references** — `document.getElementById` lookups at the top of the IIFE.
2. **Adaptive performance** — `trackPerf` averages the last 45 frame times and steps `quality` (0 full, 1 medium, 2 low) up or down.
3. **Config tables** — `DIFFICULTIES` (spawn rates, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky/cloud palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
4. **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access is direct; no try/catch wrapper exists today.
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, `panFor`. Web Audio is created lazily on the first user gesture.
6. **Rendering helpers** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw`. Canvas size is fixed at a `480×720` base (`BASE_W`, `BASE_H`) and scaled by `resize()`.
7. **Input** — keyboard (`keys` map keyed by `KeyboardEvent.code`), mouse/pointer via `canvasPointFromEvent`, and a touch joystick plus FIRE/BOMB buttons.
8. **Game state and logic** — `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `addScore`/`breakChain` (combo multiplier), `loseLife`, `endGame`, `togglePause`.
9. **Boss patterns** — `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) selected by `updateBoss`.
10. **Main loop** — `loop(now)` clamps `dt` to 50 ms, then runs `update(dt)` and `draw()` while `running && !paused`.

### Controls

- Move: mouse, arrow keys, or WASD. Fire: mouse click, Space, or the FIRE button. Bomb: `X` or Shift, or the BOMB button. Pause: `P` or Esc.
- Touch: drag in the left zone to fly, tap FIRE and BOMB.

## Conventions

- Match the existing style: 2-space indent, single quotes in JS, `const` by default, arrow functions for callbacks, and `// ---------- Section ----------` banners between sections.
- Keep the file self-contained. Do not add external scripts, stylesheets, or build tooling unless the user asks for it.
- Put tunable numbers in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Any new `localStorage` key should use the `skyfighter.` prefix and be declared alongside the existing `STORAGE_*` constants.
- Preserve the accessibility hooks: update `srLive` through `announce()` for significant events. When `reducedMotion` is on, `screenShake` is dampened (smaller magnitude, capped duration) and the hit/bomb flashes are skipped.

## Git

- Develop on the branch designated for the session; do not push to `main` without explicit permission.
