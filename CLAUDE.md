# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>` in the head), and JavaScript (one IIFE in a `<script>` block near the end of the file).
- `README.md` — placeholder; no real project docs yet.

There is no build step, package manager, bundler, test suite, or linter. Do not invent `npm` / `make` commands. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout and conventions

- Keep the game in `index.html` unless the user asks for a split. Do not add a build system without being asked.
- Everything is vanilla JS inside `(function () { 'use strict'; ... })()`. Do not leak globals and do not add modules or `import` statements.
- Section headers in the script use banner comments like `// ---------- Name ----------`. Match that style when adding a section.
- Tunable values live in top-level `const` objects near the start of the script: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, and `KILLS_PER_STAGE`. Prefer adding to these tables over hard-coding numbers in logic.
- Naming: `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constant tables, `STORAGE_*` for `localStorage` keys, and `fire*` for boss bullet-pattern helpers.
- Match the existing style: 2-space indentation, single quotes in JS, and short comments only where the logic is not obvious.

## Architecture (inside `index.html`)

Roughly in order of appearance in the script:

1. **Setup** — cached DOM references (`canvas`, HUD elements, overlay, touch controls) and `STORAGE_*` keys.
2. **Adaptive quality** — `trackPerf()` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to keep the frame rate up.
3. **Config tables** — difficulty presets, stage themes (sky gradients and sun/cloud tints), enemy types, and boss definitions.
4. **Persistence and settings** — best score, mute, difficulty, and reduced-motion. Values are stored in `localStorage` and reduced motion defaults to `prefers-reduced-motion`.
5. **Audio** — a lazily created `AudioContext` (`ensureAudio()`), with `beep()` and `noiseBurst()` building blocks and an `sfx` map of named sounds. Audio starts only after a user gesture.
6. **Input** — keyboard (`keys[]` map by `e.code`), mouse steering and click-to-fire on the canvas, and touch joystick plus FIRE/BOMB buttons.
7. **Game state and reset** — `resetGame()` initializes the player, arrays of enemies/bullets/power-ups, score, lives, bombs, weapon level, and stage.
8. **Drawing helpers** — `drawPlane`, `drawBossShip`, and `drawClouds`. Rendering is pure canvas 2D.
9. **Spawning and combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon` (Vulcan and Laser, levels 1–3), `useBomb`, and the kill-chain score multiplier (`addScore`, `breakChain`).
10. **Update loop** — `update(dt)` applies movement, firing, collisions, and stage progression. `updateBoss(dt)` drives the boss phases and patterns.
11. **Render loop** — `loop(now)` is the `requestAnimationFrame` driver. It clamps `dt` to 50 ms, samples performance, and calls `update(dt)` then `draw()` only while the game is running and not paused.
12. **Game flow** — `loseLife`, `killBoss`, `endGame`, and `togglePause`.

When changing gameplay, keep `update()` free of drawing code and `draw()` free of state mutation. The existing code mostly follows this split.

## Controls (keep in sync with the on-screen hint)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag the left zone to fly, tap FIRE, tap BOMB.

If you change a binding, update the `#controlsHint` paragraph in the HTML as well.

## Persistence keys

These `localStorage` keys are user-visible state. Do not rename them without a migration plan.

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Wrap any new storage access so the game still works when storage is unavailable (private mode, blocked site data).

## Accessibility and UX requirements

- Respect `prefers-reduced-motion`. Use the existing `reduced-motion` body class and the `reducedMotion` flag when adding screen shake, flashes, or other motion effects.
- Announce important state changes through the `announce()` helper, which writes to the `srLive` live region.
- The layout must work on phones: the game scales through `resize()` using `BASE_W` × `BASE_H` (480 × 720), and the touch controls must remain usable.

## Working on this repo

- Verify changes by loading `index.html` in a browser and playing through at least one stage, a boss fight, a bomb, and a pause/resume. There are no automated tests, so manual checks are the only verification available. Say so explicitly if you could not run the game.
- Check the browser console for errors after any change to the script.
- Keep changes focused. Do not reformat unrelated parts of `index.html`, which is a single large file and easy to churn.
- Default branch for development in this session: `claude/gifted-tesla-02kja7`. Commit with clear messages and push with `git push -u origin <branch>`. Do not open a pull request unless the user asks for one.

## Things to avoid

- Do not add external CDN scripts or fonts without a clear reason; the game is intended to be a single self-contained file.
- Do not change `BASE_W` / `BASE_H` or the canvas scaling logic without checking both desktop and touch layouts.
- Do not remove the `'use strict'` IIFE wrapper or the `Space` `preventDefault` in the keydown handler; both prevent page scrolling and global-variable leaks.
