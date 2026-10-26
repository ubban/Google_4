# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-em-up. The entire app is one file, `index.html`, which contains inline `<style>` and one inline `<script>`. `README.md` is a placeholder with no setup instructions.

There is no build step, package manager, test suite, linter, or formatter. Do not add tooling unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server. No install is needed.

To verify a change, load the page and play it. Check the title overlay, Start, and a few waves. Also check the mobile touch controls if the change touches input or layout.

## File layout

- `index.html` — markup (`#container`, `#hud`, `#touchControls`, `#overlay`), CSS (lines ~10–359), and the game script (lines ~422–1667).
- `README.md` — placeholder.
- `CLAUDE.md` — this file.

## Architecture of the game script

The script is a single IIFE with `'use strict'`. Everything is private to it, and it has no modules or globals.

- **DOM references** are grabbed once at the top by `id`. Keep new UI elements in that same pattern.
- **Config tables** sit near the top and drive balance:
  - `DIFFICULTIES` (`easy`/`normal`/`hard`): spawn timing, enemy speed, shoot chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` and `THEMES`: stage progression and sky palettes. The theme changes per stage.
  - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`): size, HP, speed, score, and the minimum stage they appear in.
  - `BOSSES`: one entry per boss, cycled through stages. `BOSS_PATTERNS` holds the attack patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, and others).
- **Main loop**: `loop(now)` runs on `requestAnimationFrame`. It clamps `dt` to 0.05 s, calls `trackPerf`, then `update(dt)` and `draw()` while the game is running and not paused.
- **`update(dt)`** handles movement, spawning, collisions, and power-ups. `updateBoss(dt)` handles boss phases. `draw()` renders everything, including the parallax clouds (`drawClouds`), planes (`drawPlane`), and bosses (`drawBossShip`).
- **Adaptive quality**: `trackPerf` averages the last 45 frames and steps a `quality` level (0 full, 1 medium, 2 low) up or down. Respect `quality` when adding expensive effects.
- **Audio** is Web Audio only, with no audio files. `ensureAudio()` must run from a user gesture. Use `beep`, `noiseBurst`, and the `sfx` helpers rather than adding new audio assets.
- **Input** covers keyboard (arrows/WASD, Space to fire, X or Shift for bomb, P or Esc to pause), mouse on the canvas, and touch (a left joystick zone plus FIRE and BOMB buttons).
- **Accessibility**: `announce()` writes to the `#srLive` region. A reduced-motion mode disables screen shake, flash, and pulse. It defaults to the `prefers-reduced-motion` media query and can be toggled with `#motionBtn`.

## Persistence

All saved state goes through `localStorage` under these keys. Keep the `skyfighter.` prefix for new keys.

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Always treat reads as possibly failing or returning `null`, and fall back to a default.

## Conventions

- Match the existing style in `index.html`: 2-space indentation, single quotes in JavaScript, `const` by default, and arrow functions for callbacks.
- Section comments use the banner form `// ---------- Name ----------`.
- Keep everything in `index.html` unless the user asks for a split. Do not add a framework or a bundler.
- Draw in the fixed `BASE_W × BASE_H` (480 × 720) canvas coordinate space. `resize()` only changes the CSS size (capped at 1.6×), so new drawing code should never assume the on-screen size.
- Keep the UI text in the overlay and HUD short. The control hints in `#controlsHint` must stay accurate when controls change.

## Git

- Develop on the branch the session names, currently `claude/gifted-tesla-hos3is`. Do not push to other branches without explicit permission.
- Write commit messages that describe the gameplay or code change, and follow the attribution rules given in the session.
