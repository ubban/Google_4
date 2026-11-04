# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file browser game: a Raiden-style vertical shoot-'em-up with stages, bosses, power-ups and smart bombs. There is no build step, no package manager and no test suite.

- `index.html` (~1,670 lines): the entire game — inline `<style>`, markup, and a single inline `<script>`. All game logic, rendering (Canvas 2D), audio (Web Audio API) and input handling live here.
- `README.md`: a placeholder; it has no setup instructions.
- Git: default working branch for feature work is `claude/gifted-tesla-fve1yq`.

## Running and verifying changes

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and load it.
- There are no automated tests or linters. Verify changes by playing the game in a browser: check the affected code path, both desktop (keyboard/mouse) and touch layouts when UI or input changes.
- Keep the file self-contained. Do not introduce external scripts, stylesheets or fonts unless the user asks for them.

## Code layout inside `index.html`

The script is organized roughly in this order (use `grep -n` to locate sections):

1. **Storage keys and persisted settings** (`STORAGE_*`): best score, mute, difficulty, reduced motion. Stored in `localStorage` under the `skyfighter.*` prefix. Reads and writes are not wrapped in try/catch; keep that in mind if you touch them.
2. **Tuning constants**: `DIFFICULTIES` (spawn rates, enemy speed, shot chance, starting lives/bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage).
3. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, plus named SFX. Audio is created lazily on first user gesture.
4. **HUD and feedback**: hearts/bombs rendering, flashes, combo pop, stage banner, screen shake.
5. **Canvas sizing**: `BASE_W = 480`, `BASE_H = 720`, scaled in `resize()`.
6. **Input**: keyboard (`keydown`/`keyup` into a `keys` map), virtual joystick and fire/bomb buttons for touch, and mouse steering and click-to-fire via `canvasPointFromEvent`.
7. **Game state and loop**: global state variables (`score`, `lives`, `stage`, `chain`, `multiplier`, etc.), `resetGame()`, and `update(dt)` as the per-frame simulation. Rendering functions (`drawPlane`, `drawBossShip`, …) sit alongside it.
8. **Spawning and combat**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, and the boss bullet-pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
9. **Performance**: `trackPerf` adjusts a `quality` level (full / medium / low) when frame time degrades.

## Conventions

- Plain vanilla JavaScript and CSS in one file. No frameworks, no transpilation, no modules.
- Keep the existing style: `const` for tuning values in `UPPER_SNAKE_CASE` objects, camelCase for functions and state.
- When adding a new enemy, boss, theme or difficulty, extend the existing table and make sure the spawn and render paths handle it.
- Respect the `reducedMotion` and `muted` settings for any new visual effects or sounds.
- Keep the layout working at phone width and preserve the `touch-action`/`user-select` rules on the game surface.
- Don't add comments that narrate obvious code; comment only non-obvious mechanics (e.g. chain/multiplier rules, boss pattern timing).

## Git workflow

- Develop on the branch the session specifies; commit with clear messages and push with `git push -u origin <branch>`.
- Don't open a pull request unless explicitly asked.
