# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static project. Its only app is **Sky Fighter**, a Raiden-style
vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

| File         | Purpose                                                        |
| ------------ | -------------------------------------------------------------- |
| `index.html` | The entire game: markup, CSS (`<style>`), and JS (`<script>`). |
| `README.md`  | Placeholder title only.                                        |

There is no build step, no package manager, no bundler, and no test suite.
Do not add `package.json`, a framework, or a build pipeline unless the user asks for one.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server
(for example `python3 -m http.server`). No install step is needed.

## Code layout inside `index.html`

- **Head / CSS** (`<style>`, roughly lines 10–360): the HUD, glassmorphic overlay card,
  touch controls, and the reduced-motion and mobile styles. Colors and layout live here.
- **Markup** (`<body>`, roughly lines 363–421): the `<canvas id="gameCanvas">`, the HUD
  elements, overlay, and touch controls. Elements are looked up by `id` in the script.
- **Script** (`<script>`, starts around line 422): one IIFE wrapping everything, with
  `'use strict'`. Nothing is exported to `window`.

Key sections of the script, in order:

1. DOM references and `STORAGE_*` keys (persistence via `localStorage`).
2. Adaptive quality (`trackPerf`), difficulty table `DIFFICULTIES`, `THEMES`,
   `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE`. Tune gameplay here first.
3. Audio: `ensureAudio`, `beep`, `noiseBurst`. Audio is synthesized with Web Audio; there are no sound files.
4. HUD helpers (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`).
5. Input: keyboard handlers (`keydown` / `keyup`), mouse and pointer handling via
   `canvasPointFromEvent`, and touch joystick (`joystickStart` / `joystickMove` / `joystickEnd`).
6. Game state: `resetGame`, `update(dt)`, `draw()`, `loop(now)`. Entities are plain objects
   in arrays (enemies, bullets, power-ups, explosions).
7. Weapons and bombs: `fireWeapon`, `useBomb`, `applyPowerUp`.
8. Bosses: `spawnBoss`, `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross,
   homing, double ring), `updateBoss`, `killBoss`.
9. Flow: `loseLife`, `endGame`, `togglePause`.

## Controls (for reference when editing input code)

- Move: arrow keys or WASD; mouse steers the ship.
- Fire: Space (hold), or hold the mouse button. Touch: tap FIRE.
- Bomb: `X` or Shift; touch: tap BOMB.
- Pause: `P` or `Esc`.
- Touch: drag on the left side to fly.

## Persistence keys

Stored in `localStorage` and read back with `Number(...)` or string comparisons.
Keep these stable so existing players keep their data:

- `skyfighter.bestScore`
- `skyfighter.muted` (`'1'` / `'0'`)
- `skyfighter.difficulty` (`easy` / `normal` / `hard`)
- `skyfighter.reducedMotion` (`'1'` / `'0'`; defaults to the `prefers-reduced-motion` media query)

Wrap new storage access in a `try/catch` if it could run in private browsing or with blocked storage.
The current code does not, so match its style only if the surrounding code is also unguarded.

## Conventions

- Match the existing style: 2-space indentation, single quotes in JS, `const` / `let`,
  arrow functions for callbacks, and section banners written as
  `// ---------- Name ----------`.
- Keep everything in `index.html` unless there is a clear reason to split it. Do not introduce
  new dependencies or CDN links without asking.
- Game timing is delta-time based (`update(dt)`). Use `dt` for movement and cooldowns rather
  than frame counts.
- Respect the reduced-motion setting when adding screen shake, flashes, or other large motion.
- Keep the layout working at phone width. The page uses a fixed `BASE_W × BASE_H` (480 × 720)
  canvas that is scaled by `resize()`.
- Accessibility: announce important state changes through the `srLive` live region via `announce()`.

## Verifying changes

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or serve it) and check the console for errors.
2. Play through at least one stage, a boss fight, a power-up pickup, a bomb, pause/resume,
   and game over, on each difficulty you touched.
3. Check the mobile layout with touch emulation if the change affects HUD or controls.
4. Confirm the best score and settings persist after a reload.

## Git

- Development branch for this work: `claude/gifted-tesla-qgqamj`. Commit and push there.
- Write clear commit messages that describe the gameplay or UI change.
- Do not open a pull request unless the user asks for one.
