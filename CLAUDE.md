# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project containing one game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 canvas.

- `index.html` — the entire game: markup, CSS (`<style>`, lines 10–359), DOM (`<body>`, lines 361–421), and a single IIFE-wrapped `<script>` (lines 422–1667) with all game logic.
- `README.md` — a stub containing only the project name and a stray timestamp. Not documentation to rely on.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Nothing needs installing.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser: start a game, check movement and firing, bombs, pause (`P`/`Esc`), stage transitions, boss fights, game over, and the best-score persistence.
- When a change touches layout, check both desktop and a narrow/touch viewport (the touch controls only appear on touch-capable devices).

## Architecture of the game script

All gameplay code lives inside one `(function () { 'use strict'; ... })();` block. Keep new code inside that scope; do not introduce globals.

Major sections, in the order they appear in the script:

1. **DOM references** — `const xxxEl = document.getElementById(...)`. If you add a UI element, add its reference here and keep the id in sync with the markup.
2. **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`. Changing a key orphans existing players' saved data, so only do it deliberately.
3. **Accessibility / perf helpers** — `announce()` (writes to the `aria-live` region) and `trackPerf()` (adaptive `quality` level 0–2 based on a rolling frame-time average).
4. **Tuning tables** — these are the main knobs for balance:
   - `DIFFICULTIES` — spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` — kills needed to advance a stage.
   - `THEMES` — background palettes, cycled per stage.
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, minimum stage 2).
   - `BOSSES` — six bosses, cycled; each has colours and an `hpMul`.
   - `BOSS_PATTERNS` (defined further down, ~line 1372) — bullet patterns used by bosses.
5. **Audio** — `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture). `sfx` holds named sound effects built from `beep()` and `noiseBurst()`. Sound must stay muteable via `muted`.
6. **Input** — keyboard state lives in `keys[e.code]`; touch uses a virtual joystick (`joystickStart/Move/End`) plus FIRE and BOMB buttons; mouse steers via `mouseTarget`.
7. **Game state and loop** — `resetGame()` initialises state; `update(dt)` advances the simulation; `draw()` renders; `loop(now)` drives both with a delta-time step. Pause is handled by `togglePause()`.
8. **Gameplay systems** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `applyPowerUp`, `useBomb`, `breakChain`/`addScore` (kill-chain score multiplier), `loseLife`, `endGame`, `killBoss`.
9. **Rendering helpers** — `drawPlane`, `drawBossShip`, `drawClouds`, explosions, screen shake, hit flash.

### Conventions to follow

- Use `dt`-based movement; do not hard-code per-frame speeds without scaling by `dt`.
- Respect `reducedMotion` (screen shake is damped and hit/bomb flashes are skipped when it is on) and `quality` when adding new visual effects.
- Every new sound goes through `sfx` and respects `muted`.
- Every new user-facing state change should call `announce()` if it matters to screen-reader users.
- Keep tuning values in the existing tables rather than scattering magic numbers.
- Keep the single-file structure unless the user asks for a split; there is no module system to rely on.

## Working in this repo

- Default branch for development: `claude/gifted-tesla-1dmk0c`. Commit and push there; do not push to other branches without explicit permission.
- Do not open pull requests unless the user explicitly asks.
- Keep changes focused. Because the whole game is one file, prefer small, surgical edits with `Edit` over rewriting large regions.
- Before finishing a gameplay change, re-read the affected section and check that no other code path (pause, game over, restart, touch input, bosses) is broken by it.

## Known gaps

- No tests, linting, or CI. Browser play-testing is the only verification.
- `README.md` is a placeholder; update it if the project's purpose or run instructions change.
