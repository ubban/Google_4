# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game is one self-contained file, `index.html`, with inline CSS and an inline `<script>`. There is no build step, no package manager, no framework, and no test suite.

`README.md` is a stub and contains no setup instructions.

## Repository layout

- `index.html` (~1,670 lines) — the whole game:
  - `<style>` (lines ~10–360): HUD, overlays, touch controls, and the vignette/flash effects. Colors and layout live here.
  - `<body>` markup (~line 363 onward): `#gameCanvas`, the HUD elements, the start/pause overlay, the touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and a `#srLive` ARIA live region.
  - `<script>` (starts ~line 422): one IIFE with `'use strict'` that holds all game state, logic, rendering, and input.
- `README.md` — project title only.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server` and then visit `http://localhost:8000`. Nothing needs to be installed.

## Architecture of the script

Everything runs inside one IIFE. Key pieces, roughly in file order:

- **DOM refs and constants.** Element lookups are at the top. `STORAGE_*` keys are prefixed `skyfighter.` (best score, mute, difficulty, reduced motion).
- **Config tables.** `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky/cloud/sun palettes for the background), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six named bosses with colors and HP multipliers). Balance changes usually belong in these tables.
- **Audio.** Web Audio synthesis (`ensureAudio`, `beep`, `noiseBurst`). No audio files. Audio is created lazily on the first user gesture (`ensureAudio()` is called from start, fire, and bomb handlers) because browsers require one.
- **Adaptive performance.** `trackPerf` samples the last 45 frame times and steps `quality` (0 full, 1 medium, 2 low) down when frames are slow and back up when they are fast.
- **Accessibility.** `announce()` writes to the `#srLive` region. Reduced motion is read from `prefers-reduced-motion` and can be overridden in-game (`reducedMotion`, `body.reduced-motion`).
- **Game state and loop.** `resetGame()` initializes state. `update(dt)` advances the simulation, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`. `dt` is clamped to 50 ms. Updates and drawing run only when `running && !paused`.
- **Weapons and bombs.** `fireWeapon()`, `useBomb()`, and `applyPowerUp()` (power-ups spawn from `maybeSpawnPowerUp`).
- **Bosses.** `spawnBoss()`, `updateBoss(dt)`, `killBoss()`, and `BOSS_PATTERNS` (radial, aimed spread, spiral, cross, homing, double ring, and similar bullet patterns).
- **Drawing.** `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. Everything is drawn to a 2D canvas. Logical size is `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()`.
- **Input.** Keyboard: Arrows or WASD move (tracked in `keys[e.code]`), `Space` fires, `KeyX` / `Shift` bombs, `KeyP` / `Escape` pauses. Mouse moves the ship over the canvas and click fires. Touch uses a virtual joystick (`#moveZone`) plus FIRE and BOMB buttons. Keys are tracked by `e.code`, not `e.key`.
- **Persistence.** Only `localStorage` is used, for best score, mute, difficulty, and reduced-motion. Every read is unguarded, so a storage-blocked browser will throw at load; keep that in mind if you touch these lines.

## Conventions

- Keep the game in a single file unless the user asks for a split. Do not introduce a bundler, framework, or npm dependencies without being asked.
- Stay in vanilla JS and the existing style: 2-space indent, single quotes in script, `const`/`let`, arrow callbacks, semicolons at statement ends.
- Put tunable numbers in the config tables near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than scattering magic numbers through the update logic.
- Preserve the `IIFE` and `'use strict'` wrapper. Do not add globals.
- Any new UI text that changes state should go through `announce()` so screen-reader users get it.
- Keep touch and keyboard controls working together. If you add an action, wire it to both.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser (or serve the folder) and check the console for errors.
2. Exercise the affected path: start a game, fire, use a bomb, pause/resume, clear a stage, and fight a boss if boss code changed.
3. Check the phone layout (narrow viewport) and touch controls if HUD, CSS, or input code changed.
4. For performance-sensitive changes, watch for frame drops; the adaptive quality system will downgrade visuals automatically.

Browser automation (Playwright, pre-installed in the cloud environment) is a reasonable way to smoke-test the game without a manual session.

## Git

- Default development branch for this session: `claude/gifted-tesla-7ux1fw`.
- Commit messages should describe the gameplay or code change in the imperative mood, matching the existing history (for example, "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
