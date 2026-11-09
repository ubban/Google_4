# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style
vertical shoot-'em-up. The entire game lives in a single file, `index.html`
(inline CSS, one inline `<script>` IIFE, no build step, no npm, no tests).

Files:

- `index.html` — the whole game (markup, styles, and logic).
- `README.md` — a stub (title only).
- `CLAUDE.md` — this file.

There is no package manifest, bundler, linter, or test suite. To run the game,
open `index.html` in a browser (or serve the directory with any static server).

## How to verify changes

No automated checks exist. After editing `index.html`:

1. Open the page in a browser and confirm there are no console errors on load.
2. Start a game, then check the affected behavior (shooting, bombs, power-ups,
   boss fights, pause, game over) at the relevant difficulty.
3. For layout or input changes, check both desktop (keyboard/mouse) and a
   narrow/touch viewport (touch controls are shown on touch devices).

Use `node --check` on the extracted script if a quick syntax check is needed.

## Code structure (`index.html`)

- **`<style>` block** — all CSS. Colors and layout for the HUD, overlay card,
  touch controls, and the `body.reduced-motion` overrides that disable screen
  shake, flashes, and pulses.
- **`<body>` markup** — `#container` holds the `#gameCanvas` canvas, the HUD
  (`#hud`: score, best, multiplier, weapon, hearts, bombs), boss bar
  (`#bossBarWrap`), `#topBar` icon buttons (mute, reduced motion, pause),
  `#touchControls`, and `#overlay` (title card, difficulty buttons, Start/Resume
  button, control hints).
- **Script** — a single `(function () { 'use strict'; ... })();` IIFE. Sections,
  in file order, roughly:
  - DOM references and storage keys (`skyfighter.bestScore`,
    `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
  - Adaptive quality (`trackPerf`) that steps `quality` 0–2 based on frame time.
  - Tuning data: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (sky palettes
    cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES`
    (six boss designs with HP multipliers).
  - Audio: a lazily created `AudioContext` (`ensureAudio`) and the `sfx` object
    built from `beep` and `noiseBurst`. Audio is created only after a user
    gesture.
  - Input: keyboard (`keys`), touch joystick and FIRE/BOMB buttons, and mouse
    steering/firing on the canvas.
  - Game state and logic: `resetGame`, `update(dt)`, `fireWeapon`, `useBomb`,
    `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`,
    `loseLife`, `endGame`, `togglePause`.
  - Boss attacks: `BOSS_PATTERNS` (one array of attack functions per boss) built
    from helpers such as `fireRadial`, `fireAimedSpread`, `fireColumns`,
    `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`.
  - Rendering: `draw()` plus `drawPlane`, `drawBossShip`, `drawClouds`.
  - Main loop: `loop(now)` calls `update` and `draw` while running and not
    paused. `dt` is clamped to 0.05 s.

## Gameplay rules (as implemented)

- **Lives and bombs** come from the difficulty table. Lives are lost on enemy
  or enemy-bullet hits, which grant 1.5 s of invulnerability and break the
  combo chain. When lives reach 0 the game ends.
- **Stages**: killing `KILLS_PER_STAGE` (18) enemies triggers a boss spawn
  about 1.2 s later. Defeating a boss adds 500 points, advances the stage, and
  changes the sky theme.
- **Weapons**: `vulcan` (spread shots) and `laser` (piercing lanes). Each weapon
  has levels 1–3. A weapon power-up raises the level; past level 3 it toggles
  to the other weapon at level 1.
- **Power-ups** dropped from destroyed enemies: `life`, `bomb`, `weapon`.
- **Score and multiplier**: kills add points through `addScore`, scaled by a
  chain multiplier that `breakChain` resets.
- **Best score** is saved to `localStorage` only when a game ends with a new
  high score.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add
  a build step, package manager, or external dependencies without asking.
- Keep the code style of the file: `const`/`let`, arrow functions, 2-space
  indentation, single quotes in JS, and short section comments with
  `// ---------- Name ----------` headers.
- Tuning values (speeds, spawn rates, HP multipliers, scores) live in the
  constant tables near the top of the script. Prefer changing those tables over
  adding magic numbers inside logic.
- Every new persisted setting should use a `skyfighter.*` key, declared as a
  constant next to the existing `STORAGE_*` keys.
- Respect reduced-motion: any new screen shake, flash, or pulse must check the
  `reducedMotion` flag and have a matching `body.reduced-motion` CSS rule.
- Keep accessibility hooks working: announcements go through `announce()` to
  the `#srLive` region, and buttons need `title` or visible text.
- Keep the layout usable at phone width. The canvas is scaled from a
  480×720 base (`BASE_W`, `BASE_H`) by `resize()`.

## Git workflow

- Develop on the branch assigned for the task (for example
  `claude/gifted-tesla-d8pf8j`). Do not push to other branches without
  permission.
- Write clear, descriptive commit messages.
- Do not open a pull request unless the user asks for one.
