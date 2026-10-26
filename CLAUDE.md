# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html` (inline CSS, inline JavaScript, no build step, no dependencies, no external network calls). `README.md` is a placeholder.

## Running it

- Open `index.html` directly in a browser, or serve the directory with any static server.
- There is no package manager, bundler, linter, or test suite configured. Verify changes by playing the game in a browser.

## File layout (`index.html`)

| Lines (approx.) | Section |
| --- | --- |
| 1–9 | `<head>`: viewport (no zoom), description, theme color, title |
| 10–360 | `<style>`: HUD, overlay, touch controls, boss bar, banners, reduced-motion styles |
| 361–421 | `<body>`: `#container` with the canvas, HUD, boss bar, touch controls, start/pause overlay |
| 422–1667 | `<script>`: the entire game, wrapped in one `(function () { 'use strict'; … })();` |

Inside the script, the code runs in roughly this order:

1. **DOM references and storage keys** — `getElementById` lookups, `STORAGE_*` constants.
2. **Feedback helpers** — `announce()` (screen-reader live region), `trackPerf()` / `quality` (auto-degrades visual effects when frames are slow).
3. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
4. **Persisted settings** — best score, mute, difficulty, reduced motion (read from `localStorage`).
5. **Audio** — lazily created `AudioContext` (`ensureAudio`), `beep()` / `noiseBurst()` primitives, and the `sfx` table of named sounds.
6. **UI helpers** — hearts/bombs rendering, flashes, combo pops, stage banner, screen shake, difficulty switching.
7. **Layout and input** — `resize()` scales a 480×720 logical canvas to the window (capped at 1.6×); keyboard, mouse, and touch (virtual joystick + fire/bomb buttons) handling.
8. **Game state** — `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score/lives/bomb/stage counters. `resetGame()` initializes them.
9. **Drawing** — `drawPlane`, `drawBossShip`, `draw()`, cloud and sky rendering per stage theme.
10. **Spawning and combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, boss attack patterns (`BOSS_PATTERNS`, indexed by boss design), `updateBoss`, `killBoss`, `loseLife`, `endGame`.
11. **Main loop** — `update(dt)` then `draw()`, driven by `requestAnimationFrame` through `loop(now)`.

## Key game rules (keep these consistent when changing code)

- **Difficulty** (`easy` / `normal` / `hard`) controls spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. It is chosen from the start screen; the difficulty buttons ignore clicks while a run is active (`setDifficulty`).
- **Stages** advance every `KILLS_PER_STAGE` (18) kills, then a boss spawns. Boss design is `BOSSES[(stage - 1) % BOSSES.length]`; theme is `THEMES[(stage - 1) % THEMES.length]`. Boss HP scales with stage (`70 + stage * 35`) and each boss's `hpMul`.
- **Enemy types** unlock by stage through `minStage` (`grunt` and `interceptor` from stage 1, `tank` from stage 2).
- **Weapons:** `vulcan` and `laser`. A weapon power-up raises the level 1→3 (more bullets/spread); past level 3 it toggles to the other weapon and resets to level 1.
- **Power-ups:** `life` (up to `maxLives`), `bomb` (up to `maxBombs`), `weapon`.
- **Combo chain:** kills feed `chain` / `multiplier`; `breakChain()` resets it.
- **Pause:** `P` or `Escape`. **Bomb:** `X` or either `Shift`. **Fire:** `Space` (held) or pointer/touch fire.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Storage access can throw in private browsing or when site data is blocked, so wrap any new `localStorage` reads and writes in `try/catch` and fall back to in-memory defaults.

## Conventions

- Plain ES2015+ JavaScript and CSS in a single file. Do not introduce a build step, framework, or external dependency without a clear reason.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes, short helper functions, and comments only where the logic isn't obvious.
- Put new tunable numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than scattering magic numbers through the update loop.
- Respect the `reducedMotion` and `quality` flags when adding particles, shake, flashes, or glow — new visual effects should degrade the same way existing ones do.
- Keep the layout working on phones (the on-screen touch controls are enabled when `isTouchDevice` is true) and keep `announce()` calls for anything a screen-reader user needs to hear.

## Working in this repo

- Develop and push changes on the branch `claude/gifted-tesla-24jxnx`.
- Keep commit messages short and descriptive.
