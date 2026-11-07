# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: inline `<style>` (~lines 10–360), markup (~361–421),
  and one inline `<script>` IIFE (~422–1670) containing all game logic.
- `README.md` — placeholder title only.

There is no build step, package manager, bundler, test suite, linter, or CI config.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server
  (e.g. `python3 -m http.server` from the repo root) and visit `http://localhost:8000`.
- There are no automated tests. Verify gameplay changes by playing in a browser, and check
  both desktop (keyboard/mouse) and touch layouts when touching HUD or controls.
- Keep the page working as a single file. Do not introduce a build step or external
  dependencies unless the user asks for it.

## Code structure (inside `index.html`'s script)

The script is one IIFE with `'use strict'`. Sections are marked with `// ---------- Name ----------`
comments. Important parts:

- **Config tables**: `DIFFICULTIES` (spawn rates, enemy speed/shoot chance, lives, bombs,
  boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`,
  `BOSSES` (name, colors, `hpMul`), and `BOSS_PATTERNS` (ordered bullet-pattern functions).
- **Persistence**: `localStorage` keys are prefixed `skyfighter.`:
  `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`,
  `skyfighter.reducedMotion`. Wrap changes to these in mind for private-mode or blocked
  storage; the existing code reads them directly.
- **Adaptive quality**: `trackPerf` samples frame time and steps `quality` (0 full, 1 medium,
  2 low) to keep the game smooth on slower devices.
- **Audio**: a lazily created Web Audio context (`ensureAudio`) with synthesized effects
  (`beep`, `noiseBurst`, `sfx.*`). Audio starts only after a user gesture.
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse steering and fire, and a touch
  joystick plus FIRE/BOMB buttons. Keyboard/joystick input takes priority over mouse.
- **Simulation**: `update(dt)` advances bullets, enemies, bosses, power-ups, and collisions.
  `dt` is clamped to 0.05 s in the loop. `draw()` renders everything to the canvas.
- **Game flow**: `resetGame`, `loseLife`, `endGame`, `togglePause`, `killBoss`, and
  `applyPowerUp`. Stage progression happens after `KILLS_PER_STAGE` kills, ending in a boss.
- **Feedback**: screen shake, hit/bomb flashes, combo popups, stage banners, and an
  `aria-live` region (`announce`) for screen readers.

Controls (also shown in the in-game hint): mouse to steer and click to fire; Arrows/WASD to
move; Space to fire; X or Shift for a bomb; P or Esc to pause. Touch: drag left to fly, tap
FIRE, tap BOMB.

## Conventions

- Keep the existing style: `const`/`let`, 2-space indentation, single quotes in JS, section
  comments, and config values in the top-level tables rather than hard-coded in logic.
- Balance changes belong in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`,
  `BOSS_PATTERNS`) where possible.
- Respect the reduced-motion setting when adding visual effects that shake, flash, or pulse.
- Any new UI control needs an accessible label or `title`, and keyboard reachability where it applies.
- Keep changes focused. Do not reformat unrelated parts of `index.html`.

## Git

- Develop on the branch specified for the task (currently `claude/gifted-tesla-ug0amd`).
  Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages that explain the why as well as the what.
- Do not open a pull request unless the user asks for one.
