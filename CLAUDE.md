# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (about 1,670 lines: inline `<style>` at lines ~10–359, markup at ~362–421, one inline `<script>` at ~422–1667). There is no build step, no bundler, no package manager, and no external dependencies.

Other files:
- `README.md` — a stub (title only).

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There is no test suite, linter, or CI configured. Verify changes by playing the game in a browser, ideally at both desktop and phone-sized viewports, since touch controls are part of the game.

## Architecture (inside `index.html`)

Everything lives in one IIFE-style `<script>` block. Major sections, roughly in source order:

1. **DOM refs and constants** (~426–466): canvas/HUD element lookups; `STORAGE_*` localStorage keys under the `skyfighter.*` prefix.
2. **Tuning data** (~485–516):
   - `DIFFICULTIES` — `easy` / `normal` / `hard`, controlling spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
   - `KILLS_PER_STAGE` (18) — kills needed before a stage boss spawns.
   - `THEMES` — six sky palettes, cycled per stage.
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP; appears from stage 2).
   - `BOSSES` — six boss designs, selected by `(stage - 1) % BOSSES.length`.
3. **Persistence and settings** (~517–541, 732–745): best score, mute, difficulty, and reduced-motion preference are read from and written to `localStorage`. Every access should stay guarded in case storage is unavailable.
4. **Audio** (~543–676): a lazily created `AudioContext` (`ensureAudio`), with helpers `beep`, `noiseBurst`, and a `sfx` table. Audio is created only after a user gesture.
5. **HUD and effects** (~680–752): `renderHearts`, `renderBombs`, flash/vignette, combo pop, stage banner, screen shake.
6. **Canvas sizing** (~753–767): `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the window, capped at 1.6×.
7. **Input** (~768–862): keyboard (Arrow keys or WASD to move, tracked in `keys[e.code]`; `Space` fires; `KeyX`/`Shift` bomb; `KeyP`/`Escape` pause), a virtual joystick and fire/bomb buttons for touch, and mouse aiming on the canvas.
8. **Game state** (~864–912): global mutable state (`player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `chain`, `multiplier`, etc.). `resetGame()` reinitializes it.
9. **Drawing primitives** (~913–1021): `drawPlane`, `drawBossShip`, and related helpers.
10. **Spawning and combat** (~1022–1147): `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `addScore` (with chain/multiplier), `useBomb` (clears enemy bullets), `fireWeapon`.
11. **Main update** (~1148–1303): `update(dt)` handles movement, collisions, stage progression, and enemy firing.
12. **Power-ups** (~1304–1323): `applyPowerUp` — `life`, `bomb`, and `weapon` (weapon cycles: vulcan level 1→2→3, then switches to laser).
13. **Boss attack patterns** (~1324–1410): `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, selected in `updateBoss` (~1411).
14. **Lifecycle** (~1441–1506): `killBoss`, `loseLife`, `endGame`, `togglePause`.
15. **Rendering** (~1507–1639): `drawClouds`, `draw()`.
16. **Loop and start** (~1640–1667): `loop(now)` drives `update` and `draw` with a delta-time clamp; the start button triggers `resetGame` and begins the loop.

Also present: a performance tracker (`trackPerf`, ~476) that sets a `quality` level (full / medium / low) when frame times degrade, and a screen-reader live region announced via `announce()`.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not introduce a build step or npm dependencies without asking.
- **No frameworks.** Plain DOM and Canvas 2D API.
- **Tuning changes belong in the data tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `KILLS_PER_STAGE`) rather than hard-coded in logic.
- **Canvas logic uses the base 480×720 coordinate space**; don't hard-code device pixels.
- **Respect the reduced-motion setting** when adding screen shake, flashes, or other motion effects (`reducedMotion`).
- **Keep touch working.** Any new control must have a touch path as well as keyboard/mouse, and `touch-action`/`user-select` are disabled on purpose.
- **Persistence keys** use the `skyfighter.` prefix. Wrap `localStorage` access so the game still runs if storage is blocked.
- **Preserve the accessibility hooks**: `announce()` for significant events, `aria-live` regions, and `sr-only` text.

## Git

- Develop on the branch named by the session (currently `claude/gifted-tesla-5daxa7`). Don't push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Commit messages: short imperative summary line, optional body explaining *why*.
