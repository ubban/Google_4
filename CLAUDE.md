# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`. The entire game (HTML, CSS, and JavaScript) lives in one file, `index.html` (~1,670 lines). `README.md` is only a title stub.

There is no build step, no package manager, no dependencies, and no test suite. Nothing is fetched from a CDN; all assets are inline (sounds are synthesized with the Web Audio API, icons are inline SVG data URIs).

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## File layout (`index.html`)

- `<head>`: meta tags (viewport locks zoom, `theme-color`), inline favicon, and a single `<style>` block. Styles cover the canvas container, HUD (`#hud`, `#hearts`, `#bombs`, stats pills), vignette/flash overlays, boss bar, overlay/menu, and touch controls.
- `<body>`: the `#container` with `<canvas id="gameCanvas">`, HUD elements, overlay (title/menu/game-over), touch controls (`#touchControls`, joystick, fire and bomb buttons), and toggle buttons (mute, motion, pause).
- One `<script>` block (starts around line 422) wrapping everything in a `'use strict'` IIFE. It caches DOM refs, then defines:
  - **Persistence keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`).
  - **Tuning tables**: `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses with HP multipliers).
  - **Audio**: lazily created `AudioContext` (`ensureAudio`), `beep`, `noiseBurst`, and the `sfx` table.
  - **Feedback**: HUD rendering, screen shake, hit/bomb flashes, combo pop, stage banner, live-region announcements for screen readers.
  - **Performance**: `trackPerf` adjusts a `quality` level (0 full, 1 medium, 2 low) from frame times.
  - **Input**: keyboard (`keys` map keyed by `e.code`), touch joystick and buttons, and mouse. Mouse and touch move the ship toward a target point.
  - **Game loop**: entities, collisions, spawning, stage progression, boss fights, and pause/game-over flow.

## Controls

- Move: Arrow keys or WASD; touch joystick on touch devices; mouse/touch drag to steer.
- Fire: Space (held), or the fire button on touch.
- Smart bomb: X or Shift, or the bomb button.
- Pause: P or Escape.

## Conventions

- **Keep it a single file** unless the user asks for a split. New code goes in the existing `<script>` block and new styles in the existing `<style>` block.
- **Game tuning belongs in the constant tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coded numbers in the loop.
- **No external network requests and no new dependencies.** Keep assets inline and audio synthesized.
- **Preserve accessibility hooks**: the `srLive` live region, `prefers-reduced-motion` handling (`reducedMotion`), and the mute/motion/pause toggles.
- **Preserve mobile behavior**: `touch-action: none`, the viewport lock, safe-area insets, and the joystick/button layout. Test changes at phone widths.
- **Use `localStorage` keys with the `skyfighter.` prefix**, and wrap storage access so the game still runs if storage is unavailable.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, and small helper functions.

## Verifying changes

There are no automated tests or linters configured. Verify by:

1. Loading `index.html` in a browser and checking the console for errors.
2. Playing through a stage transition and a boss fight, since most logic is in the main loop.
3. Checking all three difficulties and the mute/motion/pause toggles.
4. Checking layout and touch controls at a narrow (phone) viewport.

## Git

- Default development branch for automated work: `claude/gifted-tesla-psjf1n`.
- Commit messages are short imperative subject lines, optionally followed by a body explaining the why.
