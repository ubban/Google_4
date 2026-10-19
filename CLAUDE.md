# CLAUDE.md

Guidance for working in this repository.

## What this repo is

A small static project. The only real content is **Sky Fighter**, a Raiden-style
vertical shoot-'em-up written as a single self-contained HTML5 canvas page:

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>`
  wrapped in an IIFE with `'use strict'`. About 1,670 lines.
- `README.md` — a one-line placeholder. No other docs.

There is no `package.json`, no bundler, no framework, no test suite, no linter, and no
CI config. Do not add build tooling unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server
(for example `python3 -m http.server`). Nothing needs to be installed.

## Code layout inside `index.html`

- **Markup and CSS** (top of file, before `<script>`). Overlay card, HUD pills, touch
  controls, and the boss health bar are all here. Element IDs are the contract between
  markup and script, so renaming an ID means updating every `getElementById` call.
- **Script setup** (around lines 422–530). DOM references are grabbed once at the top.
  Then come the `STORAGE_*` keys, the `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, and
  `BOSSES` tables, and the adaptive-quality tracker (`trackPerf`).
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `sfx` object). Web Audio synthesis only,
  with no audio files. Audio is created lazily on the first user gesture.
- **HUD and feedback** (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`,
  `screenShake`, `flashHit`). These are presentation only.
- **Input** (keyboard, touch joystick, fire and bomb buttons, mouse aim). Keyboard
  state is stored in the `keys` map keyed by `e.code`.
- **Game state and loop** (`resetGame`, `update(dt)`, `draw()`, `loop(now)`). Game state
  lives in module-level `let`s inside the IIFE. `update` is fixed-logic simulation and
  `draw` is rendering only. Keep them separated.
- **Enemies, bosses, and power-ups** (`spawnEnemy`, `spawnBoss`, `BOSS_PATTERNS`,
  `fireRadial`/`fireCross`/etc., `maybeSpawnPowerUp`, `applyPowerUp`). Boss attack
  patterns are individual `fire*` helpers referenced from `BOSS_PATTERNS`.

## Conventions to keep

- **Persistence goes through `localStorage` with the `skyfighter.*` prefix.** Existing keys
  are `bestScore`, `muted`, `difficulty`, and `reducedMotion`. Wrap new reads and writes so
  the game still works when storage is unavailable (private mode, blocked site data).
- **Difficulty is data-driven.** Add tuning to the `DIFFICULTIES` table rather than
  hard-coding numbers in the logic.
- **Themes and bosses cycle by stage.** `THEMES[(stage - 1) % THEMES.length]` and
  `BOSSES[(stage - 1) % BOSSES.length]` are the pattern. New entries are appended to the
  arrays, not special-cased.
- **Accessibility hooks are deliberate.** Keep the `announce()` calls for state changes
  (lives, stage, game over) that feed the `srLive` live region. Respect the reduced-motion
  setting: the `body.reduced-motion` class and the `reducedMotion` flag gate screen shake
  and animations.
- **Performance.** The adaptive `quality` level (0–2) drives how much is drawn. Frame
  work should respect it, and per-frame allocations in `update`/`draw` should be avoided.
- **Mobile.** Touch controls are shown only on touch-capable devices. Keep the layout
  working at phone widths and the fixed `BASE_W × BASE_H` (480 × 720) logical canvas
  scaled by `resize()`.
- **Style.** Match the surrounding code: 2-space indent, single quotes in JS, `const` by
  default, short comment banners (`// ---------- Section ----------`) between sections,
  and no build step or external dependencies.

## Gameplay facts (for reference when changing balance)

- Stages advance after `KILLS_PER_STAGE` (18) kills, then a boss fight begins. Boss HP is
  `70 + stage * 35`, scaled by the boss's `hpMul` and the difficulty's `bossHpMul`.
- Enemy types unlock by `minStage`: grunt and interceptor from stage 1, tank from stage 2.
- Power-ups: `life` (6%), `bomb` (8%), and `weapon` (20%) of drops. The weapon power-up
  raises the level to 3, then toggles between Vulcan and Laser.
- Controls: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Escape to pause.

## Working in this repo

- Work on the branch you were given. Do not push to other branches without permission.
- Verify changes by loading `index.html` in a browser and playing a few stages. There is
  no automated test to run.
- Keep this file current when you change structure, conventions, or the game's rules.
