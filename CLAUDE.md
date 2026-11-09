# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

Files:

- `index.html` (~1,670 lines): the entire game. Inline `<style>`, markup, and one inline `<script>` IIFE. No build step, no dependencies, no external assets (the favicon is an inline SVG data URI).
- `README.md`: a one-line title only.

There is no package manager, test suite, linter, or CI configuration in the repo.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser, ideally at both desktop and phone-sized viewports, since touch controls and layout are part of the feature set.

## Code structure (`index.html`)

- **Markup/CSS (lines ~1–420):** HUD (`#hud`, hearts, bombs, weapon, multiplier), overlay card (`#overlay`), touch controls (`#touchControls`: joystick, FIRE, BOMB), boss bar, and vignette/flash effects. Colors and layout are plain CSS in the `<style>` block.
- **Script (lines ~422–end):** a single IIFE with `'use strict'`. Sections, roughly in order:
  - DOM references and `STORAGE_*` keys (`localStorage`: best score, mute, difficulty, reduced motion).
  - `DIFFICULTIES` table (easy / normal / hard): spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18) and `THEMES` (six stage palettes, each with a boss design name).
  - Audio: WebAudio `beep` / `noiseBurst` helpers and the `sfx` object, lazily initialized by `ensureAudio()` on first user gesture.
  - Input: keyboard (`keys` map), mouse (canvas move/fire), touch joystick and buttons.
  - Game state and loop: `resetGame`, `update(dt)`, `draw()`, `loop(now)`. Entities are plain arrays of objects updated each frame.
  - Weapons and power-ups: `fireWeapon`, `applyPowerUp`, `maybeSpawnPowerUp`. Weapon levels 1–3 (Vulcan/Laser).
  - Bosses: `spawnBoss`, `updateBoss`, `killBoss`, and boss bullet patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
  - Flow: `loseLife`, `endGame`, `togglePause`, `startBtn` handler.

### Controls (keep the in-game hint in sync)

- Keyboard: arrows / WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag on the left zone to fly, FIRE and BOMB buttons.

If you change a binding, update `#controlsHint` in the markup as well.

## Conventions

- Keep the game as a single self-contained file unless the user asks to split it. Don't add a bundler, framework, or external CDN dependency without being asked.
- Tune gameplay through the `DIFFICULTIES`, `KILLS_PER_STAGE`, and `THEMES` tables rather than hard-coding numbers in update logic.
- Persist player settings only through the existing `STORAGE_*` keys, and wrap new `localStorage` access in a way that tolerates it being unavailable (private browsing, blocked storage).
- Respect the reduced-motion setting when adding screen shake, flashes, or other motion effects. Follow the existing `reducedMotion` checks.
- Keep the UI usable on touch devices: no hover-only affordances, and preserve `touch-action: none` on the game surface.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for callbacks.

## Git

- Develop on the branch the session specifies (currently `claude/gifted-tesla-docn1u`). Don't push to other branches without permission.
- Commit messages in this repo use a short imperative subject line, optionally followed by a bullet-style body describing user-visible changes (see `git log`).
