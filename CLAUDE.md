# CLAUDE.md

Guidance for working in this repository.

## What this is

Google_4 is a small static web project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-em-up, implemented as a single HTML5 canvas page in `index.html` (~1,670 lines). `README.md` is a placeholder.

## Project layout

- `index.html`: the whole game. CSS is in one `<style>` block (~lines 10–360), markup in `<body>` (~361–421), and all JavaScript in one `<script>` block (~422–1667).
- `README.md`: placeholder, no setup instructions.

There is no build step, package manager, bundler, test suite, linter, or CI config. Do not add any of these unless asked.

## Running it

Open `index.html` in a browser, or serve the directory with any static server. There are no dependencies to install.

## Architecture of the game script

The script is one IIFE that starts with `'use strict'`. Keep new code inside it, so nothing leaks to `window`.

- **Setup:** DOM references are grabbed once at the top of the IIFE. Use the same pattern for new elements.
- **Persistence:** `localStorage` keys use the `skyfighter.*` prefix (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Add new keys with the same prefix.
- **Tuning data** lives in top-level constants: `DIFFICULTIES` (spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Change balance in these tables rather than in the logic.
- **Main loop:** `loop(now)` calls `update(dt)` and then `draw()`. `dt` is clamped to 50 ms. `update` owns all state changes, and `draw` only renders.
- **Adaptive quality:** `trackPerf` watches frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low). Glow effects (`shadowBlur`) are only drawn at quality 0, so keep any expensive visual behind that check.
- **Input:** keyboard (arrows/WASD to move, Space or hold to fire, X/Shift for bomb, P/Escape to pause), mouse (aim by moving over the canvas, click to fire), and touch (joystick plus FIRE and BOMB buttons). Mobile sizing is handled in `resize()`, which works from a base canvas of 480×720.
- **Accessibility:** announcements go through `announce()`, which writes to the `#srLive` live region. Reduced motion (`reducedMotion`) disables screen shake, the hit flash, and the low-health pulse. Respect it for any new motion effect.
- **Audio:** Web Audio, created lazily by `ensureAudio()` after a user gesture. Sounds are built from `beep()` and `noiseBurst()`. Mute state is stored and applied through `muted`.

## Conventions

- Stay in vanilla JavaScript and CSS with no external libraries or CDN links, so the file keeps working as a single static page.
- Match the existing style: 2-space indent, single quotes, `const` by default, short section comments of the form `// ---------- Name ----------`.
- Keep the code in one file unless the user asks for a split.

## Making changes

- After editing, test by opening `index.html` in a browser and playing through a stage. Check keyboard, mouse, and touch input where relevant, and confirm that pause, game over, and the best-score update still work. Also check the mobile layout at a narrow width.
- Keep commits focused. The existing history is a few large commits, so smaller ones will be easier to review.
- Do not commit generated or editor files. The repo has no `.gitignore` yet.
