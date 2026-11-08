# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single self-contained HTML5 canvas game. There is no build step, no package manager,
and no test suite.

- `index.html` — the entire game: inline `<style>`, markup/HUD, and one inline
  `<script>` IIFE (roughly lines 422–1669).
- `README.md` — minimal project readme.

Open `index.html` directly in a browser to play. Keep it that way unless the user
asks for a change in project structure.

## Repository layout and conventions

- Keep the game as one file. Do not split CSS/JS into separate files, add a bundler,
  or introduce a framework without an explicit request.
- No external dependencies or CDN scripts. Everything is vanilla JS and Canvas 2D.
- Code style inside the script: 2-space indent, `'use strict'`, `const`/`let`,
  arrow functions for event handlers, single quotes. Match the surrounding code.
- All game code lives inside the top-level IIFE; do not add globals.
- Section headers in the script use comment banners such as `// ---------- Input ----------`.
  Follow that pattern when adding a new section.

## Architecture of the script

Roughly in order within the IIFE:

1. **DOM references** — `getElementById` lookups for canvas, HUD, overlay, touch controls.
2. **Config tables** — `DIFFICULTIES` (spawn rates, lives, bombs, boss HP multiplier),
   `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
3. **Persistence** — `localStorage` keys prefixed `skyfighter.` (best score, mute,
   difficulty, reduced motion). Every access must be safe to fail; keep the game
   working if storage is unavailable.
4. **Audio** — lazily created via `ensureAudio()` (browsers require a user gesture).
   Sound effects are synthesized with `beep()` and `noiseBurst()`; there are no audio files.
5. **Layout** — `BASE_W = 480`, `BASE_H = 720` logical resolution; `resize()` scales
   the canvas to fit the viewport.
6. **Input** — `keys` map from `KeyboardEvent.code`, touch joystick and buttons,
   and mouse/pointer via `canvasPointFromEvent()`.
7. **Game state** — `resetGame()` initializes `player`, `bullets`, `enemies`,
   `enemyBullets`, `particles`, `powerUps`, and `boss`.
8. **Simulation** — `update(dt)` advances everything; `updateBoss(dt)` drives boss
   phases; `applyPowerUp()`, `fireWeapon()`, `useBomb()`, and the `fire*` helpers
   implement weapons and boss attack patterns.
9. **Rendering** — `draw()` renders each frame, with helpers `drawPlane`,
   `drawBossShip`, `drawClouds`, plus screen shake and flash effects.
10. **Loop** — `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05 s, and
    only updates/draws when `running && !paused`.

### Game rules to preserve

- Difficulty is chosen from `easy` / `normal` / `hard`; the chosen values feed into
  spawn rate, enemy speed and fire chance, starting lives, starting bombs, and boss HP.
- A stage ends after `KILLS_PER_STAGE` kills, followed by a boss battle.
- Weapons: Vulcan/Laser with 3 power levels. Bombs clear the screen and damage bosses.
- Score uses a kill-chain multiplier that resets via `breakChain()`.
- Adaptive quality: `trackPerf()` lowers/raises `quality` based on frame time. Effects
  should respect this level rather than hard-coding heavy work.
- Reduced motion (`body.reduced-motion`) disables or softens screen shake and flashes.
  Any new visual effect with shake or flashing must honor this flag.

## Controls (keep documentation in sync if changed)

| Action        | Keyboard                          | Touch                 |
|---------------|-----------------------------------|-----------------------|
| Move          | Arrow keys or WASD                | Left-side joystick    |
| Fire          | Space (hold for auto-fire)        | Fire button (hold)    |
| Bomb          | X or Shift                        | Bomb button           |
| Pause         | P or Escape                       | Pause button          |

## Working on this repo

- **Verify changes in a browser.** There are no automated tests, so open `index.html`
  (or serve it statically) and play through the affected path: start, fire, bomb,
  boss, pause, game over, and restart. Check the console for errors.
- Test at both desktop and phone-sized viewports when touching layout, HUD, or touch
  controls.
- Keep changes focused. Do not reformat unrelated parts of `index.html`.
- Do not add new top-level files, dependencies, or tooling unless asked.

## Git

- Default development branch for automated work: `claude/gifted-tesla-f8rymy`.
- Write descriptive commit messages in the imperative mood, matching the existing
  history style (for example, "Add Sky Fighter: ...").
- Do not open pull requests unless the user explicitly asks.
