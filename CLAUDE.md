# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a single-file, Raiden-style vertical shoot-'em-up built on an HTML5 `<canvas>`. The entire game (markup, CSS, and JavaScript) lives in `index.html`. There is no build step, no package manager, no bundler, and no test suite.

`README.md` is a placeholder and contains no setup instructions.

## Repository layout

- `index.html` — the whole game (~1,670 lines):
  - `<head>`: meta tags, an inline SVG favicon as a data URI, and an inline `<style>` block (HUD pills, overlay card, touch controls, vignette, reduced-motion rules).
  - `<body>`: the `#container` wrapper, the `#gameCanvas` canvas, HUD elements, the `#overlay` start/game-over card, the `#touchControls` layer, and the mute/motion/pause buttons.
  - A single `<script>` block that wraps everything in an IIFE. Most of the file is game logic.
- `README.md` — placeholder.
- `CLAUDE.md` — this file.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests, linters, or build commands. Verify changes by playing the game in a browser, including on a narrow/touch viewport if the change touches layout or input.
- Keep the file dependency-free. Do not add external scripts, CDNs, or a build step without an explicit request.

## Code architecture (inside the IIFE in `index.html`)

Game configuration is declared as constants near the top of the script:

- `DIFFICULTIES` — `easy` / `normal` / `hard`, each setting spawn rates, enemy speed, enemy shoot chance, starting lives, starting bombs, and `bossHpMul`.
- `KILLS_PER_STAGE` (18) — kills required before a stage boss is triggered.
- `THEMES` — six sky palettes, cycled per stage with a banner.
- `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), and `tank` (4 HP, unlocked at stage 2).
- `BOSSES` — six boss designs, selected by `(stage - 1) % BOSSES.length`. HP scales as `(70 + stage * 35) * difficulty.bossHpMul * design.hpMul`.

Core state and loop:

- Global-ish game state is declared around `let player, bullets, enemies, ...` and reset in `resetGame()`.
- `update(dt)` advances simulation, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`.
- Weapons: `fireWeapon()` implements `vulcan` and `laser`. Weapon power-ups raise `player.level` up to 3, then toggle the weapon type and reset the level to 1 (in `applyPowerUp()`). Bombs are handled by `useBomb()`.
- Boss attacks are separate pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) driven by `updateBoss(dt)`.
- Power-ups are dropped by `maybeSpawnPowerUp()` and applied by `applyPowerUp()`. Types: `life`, `bomb`, `weapon`.
- Audio is generated with the Web Audio API (`beep`, `noiseBurst`, the `sfx` table). `ensureAudio()` creates the context lazily on first user gesture.
- Rendering uses a fixed logical canvas of `BASE_W × BASE_H` (480 × 720), scaled to fit by `resize()`.

Persistence (`localStorage`), all keys prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Keep these key names stable; changing them silently resets players' saved data.

## Input model

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause. Space's default is prevented.
- Mouse: move to steer, click to fire.
- Touch: drag in the left zone for a virtual joystick, tap FIRE, tap BOMB. `touch-action: none` is set on the page.

Any new control must work on all three input paths, or be explicitly scoped to one.

## Conventions

- Keep the single-file structure unless the user asks for a split.
- Use `const`/`let`, the existing 2-space indentation, and single quotes in JS.
- Game tuning values live in the constant tables above. Prefer adding to those tables over scattering magic numbers through the loop.
- Respect `reducedMotion`: new screen shake, flashes, or animations should check the existing flag and degrade gracefully.
- Wrap any new `localStorage` access in a `try`/`catch` if it could run where storage is blocked, matching the spirit of the existing code.
- Performance: `trackPerf()` samples the last 45 frame times and steps `quality` (0 full, 1 medium, 2 low) down when frames are slow and back up when they are fast. Glow, particle counts, and shadow blur key off this value. Avoid per-frame allocations in hot paths (`update`, `draw`, boss patterns) where practical.

## Git

- Develop on the branch the session specifies; do not push to `main` without permission.
- Write descriptive commit messages in the imperative mood.
