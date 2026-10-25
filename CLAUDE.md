# CLAUDE.md

Guidance for working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- `index.html` is the entire app: inline CSS (`<style>`, roughly lines 10–360), the markup and HUD (`<body>`, roughly lines 361–421), and one inline `<script>` IIFE (roughly lines 422–end) that holds all game logic.
- `README.md` is a placeholder and has no setup instructions.
- There is no build step, no package manager, no bundler, and no test suite. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server` from the repo root. Test changes by playing in a real browser. Check desktop keyboard controls, the touch overlay (use device emulation), and the audio toggle.

## Code layout inside the script

The script is one `(function () { 'use strict'; ... })()` closure. Nothing is exported to `window`, so keep new code inside it.

Major sections, in the order they appear:

1. **DOM refs and constants**: `getElementById` lookups, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage, cycled), `ENEMY_TYPES`, `BOSSES`.
2. **Persistence**: best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under the `skyfighter.*` keys. Every access goes through `localStorage` directly, so keep reads and writes wrapped if you add more.
3. **Adaptive quality**: `trackPerf` samples frame time and steps `quality` between 0 (full), 1 (medium), and 2 (low) to keep frame rate up.
4. **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` helpers. Audio starts only after a user gesture.
5. **HUD and feedback**: hearts, bombs, combo pop, stage banner, screen shake, hit flash, and `announce()` for the screen-reader live region.
6. **Input**: keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and touch joystick and fire/bomb buttons.
7. **Game state**: `resetGame()` sets the per-run variables (`score`, `lives`, `stage`, `kills`, and so on). `update(dt)` advances the simulation. `draw()` renders. `loop(now)` drives both from `requestAnimationFrame`.
8. **Enemies, bosses, power-ups**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, and the boss attack patterns in `BOSS_PATTERNS`.

## Gameplay rules to preserve

- Lives and bombs depend on difficulty (`DIFFICULTIES`). Easy, normal, and hard differ in spawn rate, enemy speed, enemy fire chance, and boss HP multiplier.
- A stage ends after `KILLS_PER_STAGE` kills and then starts a boss fight. Stages advance the theme and boss.
- Power-up drops are random (`maybeSpawnPowerUp`): extra life, bomb, and weapon upgrade.
- Score uses a kill-chain multiplier (`addScore`, `breakChain`).
- Best score is saved to `localStorage` only when a run ends with a new high (`endGame`).

## Controls

- **Keyboard:** arrow keys or WASD to move, Space to fire, X or Shift to use a bomb, P or Escape to pause.
- **Mouse:** hovering over the canvas moves the ship toward the cursor. Holding the mouse button down on the canvas fires.
- **Touch:** virtual joystick on the left, fire and bomb buttons on the right.

## Conventions

- Plain browser JavaScript with `const` and `let`, arrow functions, and no modules. Match the existing style: two-space indent, single quotes, short section comments with `// ---------- Name ----------`.
- Do not add frameworks, build tools, or external dependencies. Keep the game a single self-contained file unless the user asks otherwise.
- Keep the `skyfighter.*` `localStorage` key names stable. Changing them would wipe players' saved best score and settings.
- Preserve accessibility hooks: the `srLive` announcements, the reduced-motion toggle, and the `prefers-reduced-motion` check.
- Preserve the `pointer`/`touch` split. Touch controls should not break mouse or keyboard play.

## Git

- Development branch for this work: `claude/gifted-tesla-swtdmz`.
- Commit messages are short imperative summaries. Look at `git log` for the house style.
