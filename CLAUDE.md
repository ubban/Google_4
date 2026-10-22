# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up that runs in the browser. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: markup, CSS (`<style>`, ~line 10), and JavaScript (`<script>`, ~line 422). All game code lives in one IIFE.
- `README.md` — minimal project title only.

To run the game, open `index.html` in a browser (or serve the directory with any static server). There is nothing to install.

## Repository layout

```
index.html   # full game (HTML + CSS + JS)
README.md
CLAUDE.md    # this file
```

Keep the game in a single file unless the user asks for a split. Do not add a bundler, framework, or dependencies without asking.

## Code structure inside `index.html`

The script is wrapped in an IIFE and organized roughly in this order:

1. **DOM references and storage keys** — `getElementById` lookups; `STORAGE_*` constants for `localStorage` keys (`skyfighter.*`).
2. **Perf / quality scaling** — `trackPerf` adjusts a `quality` level (0 full, 1 medium, 2 low) from average frame time. Particle counts and glow respond to it.
3. **Config tables** — `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (per-stage boss designs).
4. **Settings** — best score, mute, difficulty, and reduced-motion preference, persisted to `localStorage`.
5. **Audio** — lazily created `AudioContext` (`ensureAudio`), with `beep`/`noiseBurst` primitives and a named `sfx` table. Audio must start from a user gesture.
6. **Feedback FX** — hearts/bombs HUD rendering, hit/bomb flashes, combo pop, stage banner, screen shake.
7. **Layout and input** — `resize()` scales a fixed 480×720 logical canvas to the window. Keyboard (`keys`), touch joystick and buttons, and mouse steering all write to shared state (`firing`, `joyVec`, `mouseTarget`).
8. **Game state** — `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score, lives, bombs, chain/multiplier, stage, and kill counters.
9. **Weapons** — `fireWeapon()` (levels 1–3 with spread/lanes), `useBomb()`, and `applyPowerUp()`.
10. **Boss patterns** — `BOSS_PATTERNS` maps each boss design to attack functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` cycles them and handles telegraphs.
11. **Update loop** — `update(dt)` handles movement, spawning, collisions, and stage progression. `killBoss()` increments `stage`; `loseLife()` and `endGame()` handle failure.
12. **Rendering** — `draw()` renders the sky gradient, clouds, sprites (`drawPlane`, `drawBossShip`), and particles. Screen shake offsets the whole frame.
13. **Main loop** — `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` when running and not paused.

### Game flow

- Stage progression: after `KILLS_PER_STAGE` kills, a boss warning fires and `spawnBoss()` runs ~1.2 s later. Defeating the boss advances `stage`, which changes the theme, boss design (`(stage - 1) % BOSSES.length`), and enemy mix (`ENEMY_TYPES[*].minStage`).
- Difficulty sets spawn rate, enemy speed, shot chance, starting lives and bombs, and boss HP multiplier.
- Chain/combo raises the score multiplier; losing a life or taking damage breaks the chain.

## Conventions

- **Match the existing style**: 2-space indent, single quotes in JS, `const` by default, short arrow functions for handlers, and terse inline comments only where the logic is not obvious.
- **Names**: `camelCase` for functions and variables, `SCREAMING_SNAKE_CASE` for config constants and storage keys.
- **Game-balance numbers** live in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`). Prefer changing those over hard-coding new magic numbers in update logic.
- **Persistence**: any new saved setting uses a `skyfighter.*` key and must be read and written inside a `try`-safe path, since `localStorage` can throw in private or restricted contexts. The existing code does not guard it yet; do not make that worse.
- **Accessibility**: respect `reducedMotion` for shake, flashes, and pulses. Announce important state changes through the `srLive` live region (`announce()`).
- **Mobile**: keep touch controls working (`touchControls`, joystick, FIRE/BOMB). Layout must hold at phone width.

## Verifying changes

There is no automated test or lint setup. To verify a change:

1. Open `index.html` in a browser and check the console for errors.
2. Play through at least one stage transition and one boss fight if you touched update, boss, or stage logic.
3. Check all three difficulties if you changed balance values.
4. Check touch controls in a mobile-sized viewport if you touched input or layout.

If you add a test or build tool, document the command here.

## Git

- Develop on the branch the session specifies (currently `claude/gifted-tesla-1xsmum`). Do not push to other branches.
- Commit with clear, descriptive messages. Do not open a pull request unless explicitly asked.
