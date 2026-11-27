# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single self-contained file:

- `index.html` (~1,670 lines): all markup, CSS (`<style>`, starts ~line 10), and game logic (one inline `<script>`, starts ~line 422, wrapped in an IIFE with `'use strict'`).
- `README.md`: placeholder title only.

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. The game is plain HTML5 Canvas 2D plus Web Audio.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

There are no install or build commands to run.

## Architecture

All game code lives in the single IIFE. Key sections, roughly in file order:

| Area | What it does |
| --- | --- |
| DOM refs (top of script) | `getElementById` lookups for canvas, HUD, overlays, touch controls. |
| Storage keys | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`. |
| Adaptive performance | `trackPerf(dt)` samples the last 45 frames and steps a `quality` level (0 full, 1 medium, 2 low) when average frame time drifts. |
| `DIFFICULTIES` | Per-difficulty tuning: spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier. Values are `easy`, `normal`, `hard`. |
| `KILLS_PER_STAGE` | Kills needed to advance a stage (18). |
| `THEMES` | Six sky palettes, cycled per stage. |
| `ENEMY_TYPES` | `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+). |
| `BOSSES` | Six boss definitions (colors, name, HP multiplier), cycled. |
| Audio | `ensureAudio()` lazily creates the `AudioContext`. `beep()`, `noiseBurst()`, and the `sfx` object synthesize all sound. No audio files. |
| Input | Keyboard (Arrow keys or WASD to move via `keys[e.code]`, Space fire, `KeyX`/Shift bomb, `KeyP`/Escape pause), mouse on canvas, and touch joystick plus fire/bomb buttons. |
| Scaling | `BASE_W = 480`, `BASE_H = 720` logical resolution; `resize()` fits the canvas to the viewport. |
| Game state | `resetGame()`, `update(dt)`, `draw()`. Entities are plain objects in arrays: `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, plus `boss`. |
| Weapons and patterns | `fireWeapon()` (player, levels up to 3), and boss bullet patterns `BOSS_PATTERNS` (radial, spread, spiral, cross, homing, etc.). |
| Power-ups | `life`, `bomb`, `weapon`, applied in `applyPowerUp()`. |
| Loop | `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw` when running and not paused. |

### Game flow

Start button → `resetGame()` → `update`/`draw` loop. Losing a life calls `loseLife()`, which ends the run in `endGame()` when lives run out. Pausing goes through `togglePause()`. A boss spawns (via `spawnBoss()`, ~1.2 s after the stage's kill quota of `KILLS_PER_STAGE` is reached); boss behavior is in `updateBoss()`, and defeat is handled in `killBoss()`.

### Accessibility and persistence

- Reduced motion: the `reducedMotion` flag (from the `prefers-reduced-motion` media query, overridable via the motion button and `localStorage`) shrinks screen shake and suppresses the hit flash. Keep this in mind for any new visual effect.
- Screen-reader updates go through `announce(text)`, which writes to the `srLive` live region.
- Best score, mute, difficulty, and motion preference persist in `localStorage`. Access to `localStorage` can throw in some browsers, so keep reads/writes defensive if you add new ones.

## Conventions

- **Keep it a single file** unless the user asks for a split. Styles stay in `<style>`, logic in the single `<script>` block.
- **Vanilla JS only.** Do not add frameworks, bundlers, or CDN dependencies without explicit approval.
- **Match the existing style:** 2-space indentation, `const`/`let` (no `var`), arrow functions for callbacks, section comments in the form `// ---------- Name ----------`.
- **Frame-rate independence:** movement and timers use `dt` (seconds). Multiply per-frame motion by `dt`; do not assume 60 fps.
- **Canvas coordinates** are in the `BASE_W × BASE_H` logical space, not CSS pixels.
- **Audio** is synthesized. New sounds should be added to the `sfx` object using `beep` or `noiseBurst`, not as asset files.
- Use `Number(...) || default` style for numeric `localStorage` values, as the existing code does.

## Working in this repo

- Development for this session happens on branch `claude/gifted-tesla-onxq9q`.
- There are no tests or linters configured. Verify changes by loading `index.html` in a browser and playing through the relevant path (start, shoot, pause, bomb, game over, difficulty change, mute). For visual or gameplay changes, a headless browser check is the most reliable way to confirm nothing broke.
- When changing balance numbers, edit the constants in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES` rather than hard-coding values in logic.
- When adding a new enemy, boss, power-up, or pattern, follow the existing entries in the matching table and wire it into `spawnEnemy` / `spawnBoss` / `maybeSpawnPowerUp` / `applyPowerUp` / `BOSS_PATTERNS`.

## Known gaps

- `README.md` is just a title line with a stray timestamp; it does not describe the game. Update it if the project needs proper documentation.
- No automated tests exist.
