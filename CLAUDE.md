# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static project containing **Sky Fighter**, a Raiden-style vertical shoot-em-up
written as a single self-contained HTML5 canvas app.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>`
  (~1,700 lines). No build step, no bundler, no dependencies.
- `README.md` — a stub (title plus a timestamp line). Not user-facing documentation.

There is no package manager, test suite, linter, or CI configuration in the repo.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server
(for example `python3 -m http.server` from the repo root) and visit it on localhost.

There is nothing to install or compile. To verify a change, load the page and play it;
the canvas is 480×720 base resolution and scales to fit the window (capped at 1.6×).

## Code layout inside `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Within it, roughly in order:

1. **DOM references** — `getElementById` lookups at the top of the script. If you add a
   new HUD or overlay element, add its lookup here.
2. **Storage keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`,
   `skyfighter.difficulty`, `skyfighter.reducedMotion`).
3. **Tuning tables** — `DIFFICULTIES`, `THEMES` (six background palettes cycled by stage),
   `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses, one per stage cycle).
   Balance changes usually belong in these tables rather than in logic.
4. **Audio** — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object).
   No audio files are loaded.
5. **HUD helpers** — hearts, bombs, combo pop, stage banner, screen shake, flash.
6. **Input** — keyboard (`keys` map), touch joystick and fire/bomb buttons, and mouse
   drag on the canvas.
7. **Game state and logic** — `resetGame`, `update(dt)`, spawning, collisions, power-ups
   (`applyPowerUp`), bombs (`useBomb`), weapons (`fireWeapon`), boss patterns
   (`BOSS_PATTERNS`, `updateBoss`), and `loseLife` / `endGame`.
8. **Rendering** — `draw()` and the sprite helpers (`drawPlane`, `drawBossShip`, clouds).
9. **Main loop** — `loop(now)` with a fixed-ish `dt`, plus an adaptive quality tier
   (`trackPerf`) that lowers visual detail when frame times slip.

## Game facts worth knowing before editing

- **Controls:** arrows or WASD to move; Space or hold-fire to shoot; X / Shift for bomb;
  P or Escape to pause. Touch devices get an on-screen joystick, fire, and bomb.
- **Difficulty** (easy / normal / hard) changes spawn rate, enemy fire chance, lives,
  starting bombs, and boss HP multiplier.
- **Stages** advance every `KILLS_PER_STAGE` (18) kills and end in a boss fight with a
  health bar.
- **Weapons:** vulcan and laser, each with 3 levels; a weapon power-up raises the level,
  then switches weapon at max level.
- **Score** uses a kill-chain multiplier (`addScore`, `breakChain`).
- **Persistence** is limited to `localStorage` (best score, mute, difficulty, reduced
  motion). Every access should stay tolerant of storage being unavailable or disabled.
- **Accessibility:** a `reducedMotion` setting (defaulting to the OS
  `prefers-reduced-motion` preference) disables shake and similar effects, and an
  `aria-live` region (`srLive` / `announce`) reports key events.

## Conventions

- Keep it a single file unless the user asks for a restructure. Don't introduce a build
  step, framework, or external dependency to make a small change.
- Match the existing style: `const`/`let`, arrow functions for callbacks, two-space
  indentation, and section banners written as `// ---------- Name ----------`.
- Prefer editing the data tables and constants over adding special cases in logic.
- When adding a persisted setting, add a `STORAGE_*` key next to the others and read it
  with a fallback default.
- Don't add a `<script src>` or `<link>` to external CDNs without a clear reason; the game
  currently works fully offline.

## Git

- Default development branch for this session: `claude/gifted-tesla-b7t4rk`.
- Commit messages in this repo use a short imperative subject followed by a bulleted body
  describing features, as in the existing `Add Sky Fighter` commit.
- Don't open pull requests unless explicitly asked.
