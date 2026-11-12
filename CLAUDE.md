# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines: inline CSS, markup, and one IIFE-wrapped `<script>`). There is no build step, no package manager, no tests, and no server-side code.

Files:

- `index.html` — the whole game (markup, styles, and logic).
- `README.md` — placeholder; contains only a title and a stray timestamp.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). Nothing needs to be installed.

## Architecture of `index.html`

Layout, top to bottom:

1. `<style>` (≈ lines 10–360): all CSS, including the `reduced-motion` body class and touch-control layout.
2. `<body>` (≈ line 361): a `<canvas id="gameCanvas">` plus HUD, overlay, pause/mute/motion buttons, touch controls, and a boss health bar. Every element the script touches is looked up by `id` at the top of the script.
3. One `<script>` containing a single `(function () { 'use strict'; ... })();` IIFE. Nothing is exported or attached to `window`; keep it that way unless there is a reason to change it.

Major sections inside the IIFE, in source order:

| Section | Purpose |
| --- | --- |
| Config constants | `DIFFICULTIES` (spawn rates, speeds, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses with colors and HP multipliers), `BOSS_PATTERNS` |
| Persistence | `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion` |
| Audio | Lazily created `AudioContext` (`ensureAudio`), synthesized SFX via `beep` / `noiseBurst`; no audio files |
| Input | Keyboard (arrow keys or WASD to move, Space to fire, `X`/Shift to bomb, `P`/Escape to pause), mouse on canvas, and touch joystick + fire/bomb buttons |
| Game state | `resetGame()` initializes `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` (types: `life`, `bomb`, `weapon`) |
| Simulation | `update(dt)` drives movement, collisions, scoring, combos, and power-up pickup; `updateBoss(dt)` runs boss attack patterns |
| Rendering | `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`; screen shake via `screenShake` |
| Loop | `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 50 ms, and calls `trackPerf` |

### Notable mechanics

- **Adaptive quality:** `trackPerf` averages the last 45 frame times and steps a `quality` level (0 full, 1 medium, 2 low) down when the frame rate drops below ~40 fps and back up above ~56 fps.
- **Stages:** every `KILLS_PER_STAGE` (18) kills advances the stage, which switches the theme and eventually brings a boss. Bosses cycle through `BOSSES`.
- **Scoring:** `addScore` applies a combo multiplier; `breakChain` resets it on damage.
- **Accessibility:** `announce()` writes to an `aria-live` region; `reducedMotion` defaults to the `prefers-reduced-motion` media query and can be overridden by the user.
- **Randomness:** enemy placement, boss attack patterns, and particles all use `Math.random()` directly. There is no seeded RNG, so runs are not reproducible.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a bundler, framework, or npm dependencies without asking.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, `let` only when reassigned, arrow functions for callbacks, and section headers written as `// ---------- Name ----------`.
- Tuning values (speeds, spawn rates, HP multipliers, scores) live in the config tables near the top of the script. Prefer changing those over hard-coding numbers in logic.
- Anything written to `localStorage` must use the `skyfighter.` prefix and must be wrapped so the game still works when storage is unavailable or the value is malformed.
- Preserve the `prefers-reduced-motion` behavior and the `aria-live` announcements when adding new visual effects or messages.
- Keep `BASE_W = 480`, `BASE_H = 720` as the logical play area; `resize()` scales the canvas to fit the viewport.

## Verifying changes

There is no test suite or linter. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Play through at least one stage on each difficulty when changing gameplay, and confirm pause/resume, mute, motion toggle, and best-score persistence still work after a page reload.
3. Check touch controls with a touch-emulated viewport when changing input or layout.
4. For CSS or layout changes, check a phone-width viewport and a desktop viewport.

Be honest in summaries about what was and was not exercised; a successful page load is not the same as a verified gameplay change.

## Git workflow

- Default working branch for this session: `claude/gifted-tesla-5rhskq`. Do not push to other branches without explicit permission.
- Commit messages should be short imperative summaries. Do not create a pull request unless the user explicitly asks for one.
