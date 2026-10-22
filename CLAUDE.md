# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up.

- `index.html` is the entire game: markup, CSS (`<style>`, ~lines 10–362), and one IIFE-wrapped `<script>` (~lines 422–1667) with no external libraries or build step.
- `README.md` is a placeholder with no setup instructions.

There is no package manager, test suite, linter, or build. To run the game, open `index.html` in a browser.

## Working on the code

- Keep it a single self-contained file unless the user asks for a split. Do not add a bundler, framework, or npm dependencies without asking.
- Follow the existing style: `const`/`let`, `function` declarations inside the IIFE, `'use strict'`, two-space indentation, single quotes, and short section comments such as `// ---------- Adaptive performance ----------`.
- Everything lives inside `(function () { ... })()`. Nothing is exposed on `window`, so do not reference game internals from outside the script.
- Verify changes by loading `index.html` in a browser (or Playwright against the pre-installed Chromium) and playing through the affected path. There are no automated tests.

## Structure of the game script

Roughly in file order:

| Area | Notes |
| --- | --- |
| DOM lookups | Cached `const` references at the top of the IIFE. Add new element references here. |
| Storage keys | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`. Reads and writes are not wrapped in try/catch today. |
| Adaptive quality | `trackPerf()` samples frame time and steps `quality` 0–2 (full / medium / low) to shed effects on slow devices. |
| Config tables | `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `THEMES` (per-stage sky palette, cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank, with `minStage` gating), `BOSSES` (six bosses, one per stage cycle), `KILLS_PER_STAGE = 18`. Tune balance here first. |
| Audio | Web Audio synthesis only (`beep`, `noiseBurst`, `ensureAudio`). No audio files. Mute state is persisted. |
| Layout | `BASE_W = 480`, `BASE_H = 720` logical canvas size; `resize()` scales to the viewport. |
| Input | Keyboard (arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause), mouse on the canvas, and on-screen joystick plus fire and bomb buttons for touch. |
| Game state | `resetGame()` initializes state. `update(dt)` runs the simulation, `draw()` renders, `loop(now)` drives both via `requestAnimationFrame` with `dt` clamped to 50 ms. |
| Weapons and power-ups | `fireWeapon()` (Vulcan/Laser, 3 levels), power-up types `life`, `bomb`, `weapon` created in `maybeSpawnPowerUp()` and applied in `applyPowerUp()`. |
| Bosses | `BOSS_PATTERNS` are bullet-pattern functions (radial, spiral, cross, homing, etc.). `updateBoss()` picks patterns; `killBoss()` and `loseLife()` handle transitions. |
| Game flow | `endGame()` updates best score and stage stats; `togglePause()` handles pause. |

## Gameplay rules to preserve

- Kill chains raise the score multiplier. `breakChain()` resets it when the chain timer runs out and when the player loses a life.
- Bombs (`useBomb()`) clear enemy bullets and on-screen enemies, and deal 24 damage to a boss.
- Lives and bombs come from the difficulty table. Extra lives and bombs can also drop as power-ups.
- Every `KILLS_PER_STAGE` kills triggers a boss fight with a health bar (`bossBarWrap`). Boss designs cycle through `BOSSES` by stage, and each stage uses the matching `THEMES` entry.
- Reduced motion (`skyfighter.reducedMotion`, defaulting to the OS `prefers-reduced-motion` setting) suppresses the hit and bomb flashes.

## Conventions

- Keep UI text user-facing and accessible: announce important state changes through `announce()`, which writes to the `srLive` live region.
- When adding a new enemy, boss, theme, or power-up, add it to the existing config table and its handler rather than hard-coding special cases elsewhere.
- Changes that touch the HUD, overlay, or touch controls should be checked at phone width as well as desktop width.

## Git

- Development happens on the branch named in the session instructions. Do not push to `main` directly.
- Commit messages in this repo are descriptive and use a short summary line followed by bullet points for details.
