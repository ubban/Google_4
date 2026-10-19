# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs entirely in the browser. The whole game is a single self-contained file, `index.html`, with inline CSS and one inline `<script>` block. There is no build step, no bundler, no package manager, and no test suite.

`README.md` is a placeholder and carries no project documentation.

## Repository layout

```
.
├── index.html   # The entire game: markup, CSS (inline <style>), and JS (inline <script>)
└── README.md    # Placeholder
```

Inside `index.html`, the layout is:

- `<head>` (about lines 3–9): meta tags, title, and an inline SVG favicon as a data URI.
- `<style>` (about lines 10–360): all styling, including the `reduced-motion` body class and touch-control layout.
- `<body>` (from about line 361): the `#gameCanvas` element, the HUD, overlays, and touch controls.
- `<script>` (from about line 422): one IIFE wrapping all game logic, with `'use strict'`.

Line numbers drift as the file changes. Use `Grep` for function names rather than trusting these numbers.

## Running the game

No install is needed. Open the file directly in a browser:

```
xdg-open index.html        # Linux
open index.html            # macOS
```

Or serve the directory with any static server, for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000`.

## Architecture of the game script

All state lives in closure variables inside the IIFE. Nothing is exported or attached to `window`.

- **Canvas and scaling.** The logical resolution is fixed at `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas CSS size to fit the window, capped at 1.6×. Drawing always happens in the base coordinate space.
- **Main loop.** `loop(now)` runs on `requestAnimationFrame`. It clamps `dt` to 0.05 s, calls `trackPerf`, then calls `update(dt)` and `draw()` only when `running && !paused`.
- **Adaptive quality.** `trackPerf` keeps a 45-frame window of frame times and steps the `quality` level (0 full, 1 medium, 2 low) up or down based on average frame time.
- **Difficulty.** `DIFFICULTIES` holds per-level tuning (spawn rates, enemy shoot chance, lives, bombs, boss HP multiplier). `setDifficulty` applies it.
- **Content tables.** `THEMES` (background palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses, one per stage cycle). `KILLS_PER_STAGE` controls stage progression. Boss attacks are defined in `BOSS_PATTERNS` (radial, spread, spiral, cross, homing, double-ring).
- **Entities.** Enemies, bullets, power-ups, and explosions are plain arrays of objects, mutated in `update()`. `rectsOverlap` handles collision.
- **Rendering.** `draw()` paints everything in one pass. Helpers include `drawPlane`, `drawBossShip`, and `drawClouds`.
- **Audio.** Synthesised with the Web Audio API through `ensureAudio()`, `beep()`, and `noiseBurst()`. There are no audio files. `sfx` wraps the named sounds.
- **Input.** Keyboard uses `e.code`, so layout-independent key codes are required. Touch uses a virtual joystick and fire/bomb buttons. Canvas pointer events are converted to base coordinates by `canvasPointFromEvent`.
- **Persistence.** `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access must stay safe if storage is unavailable or throws.
- **Accessibility.** `announce()` writes to the `#srLive` live region. `reducedMotion` (from the OS preference or the user toggle) softens screen shake and suppresses the hit and bomb flashes.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce a build step or external dependencies without a clear reason.
- **No external network calls.** The page loads nothing from a CDN. Keep it that way. The favicon is an inline data URI.
- **Style.** Match the existing code: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short section comments in the form `// ---------- Name ----------`.
- **Naming.** Constants are `UPPER_SNAKE_CASE` (`BASE_W`, `KILLS_PER_STAGE`, `STORAGE_BEST`). Functions are `camelCase` verbs (`spawnEnemy`, `fireWeapon`, `endGame`).
- **Colors.** Enemy and boss colors are hex strings in their config objects. Theme cloud and sun colors are `'r,g,b'` strings, used in `rgba()` calls.
- **Balance changes.** Tuning lives in the config tables at the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer editing those over hard-coding numbers in logic.
- **Keep the mobile layout working.** The page is used on phones. Test any HUD or overlay change at narrow widths, and keep touch controls usable.

## Verification

There are no automated tests or linters configured. Verify changes by:

1. Opening `index.html` in a browser and playing a few stages on the affected difficulty.
2. Checking the browser console for errors. A clean run should produce none.
3. For input changes, testing keyboard, pause (`P`/`Esc`), bomb (`X`/`Shift`), and, if relevant, touch controls.
4. For rendering changes, checking at a narrow viewport and with `prefers-reduced-motion` enabled.

If a change touches only comments or docs, a quick syntax sanity check of the script block is enough.

## Git

- Develop on the branch specified by the session. Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless the user asks for one.
