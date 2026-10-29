# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- The entire app lives in `index.html` (inline `<style>`, markup, and one IIFE `<script>`). There is no build step, no bundler, no package manager, and no test suite.
- `README.md` is a placeholder and contains no setup instructions.
- To run the game, open `index.html` in a browser. Nothing needs to be installed.

## Repository layout

```
index.html   # The whole game: CSS, DOM, and JavaScript
README.md    # Placeholder
```

## Code structure inside `index.html`

The script is wrapped in a single `(function () { 'use strict'; ... })()`. Keep it that way so nothing leaks onto `window`.

Major sections, in file order:

- **DOM refs and storage keys**: `STORAGE_*` constants use the `skyfighter.*` prefix in `localStorage`.
- **Adaptive performance** (`trackPerf`): tracks frame times and lowers or raises a `quality` level (0 full, 1 medium, 2 low).
- **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six designs, picked by `(stage - 1) % BOSSES.length`), `BOSS_PATTERNS` (bullet patterns).
- **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, `sfx.*`). Audio is created lazily on the first user gesture.
- **Input**: keyboard (`keys[e.code]`), mouse on the canvas, and touch joystick plus FIRE and BOMB buttons.
- **Game state**: module-level `let`s declared near `resetGame()`. `resetGame()` is the single place that initializes a run.
- **Simulation**: `update(dt)`, with helpers for spawning, weapons, power-ups, and boss logic.
- **Rendering**: `draw()` and the `draw*` helpers. Rendering is separate from simulation.
- **Loop**: `loop(now)` runs a variable-timestep update (`dt` clamped to 0.05s) and then `draw()`.

## Game rules worth knowing before changing things

- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy shot chance, lives, starting bombs, and boss HP multiplier.
- A stage ends after `KILLS_PER_STAGE` (18) kills. That triggers `spawnBoss()` about 1.2s later. Killing the boss advances `stage` and resets `stageKills`.
- Power-ups (`life`, `bomb`, `weapon`) are dropped by enemies and applied in `applyPowerUp`. The weapon power-up raises `player.level` up to 3, then toggles between `vulcan` and `laser`.
- A kill chain drives the score multiplier (`addScore`, `breakChain`).
- Bombs (`useBomb`) clear the screen and damage bosses. They are triggered with X or Shift on keyboard and the BOMB button on touch.
- Best score, mute, difficulty, and reduced-motion preference persist in `localStorage`. Reads and writes must tolerate storage being unavailable.

## Conventions

- Match the existing style: 2-space indentation, single quotes, `const` by default, arrow functions for callbacks, and short section comments with `// ---------- Name ----------`.
- Keep new tuning values in the existing constant tables rather than hard-coding them in logic.
- Keep the game playable on touch devices and at phone widths. The canvas is scaled to fit, with `BASE_W = 480` and `BASE_H = 720` as the logical size.
- Respect `reducedMotion` for any new screen shake, flashes, or particle-heavy effects.
- Do not add external dependencies or CDN scripts. The game is intentionally self-contained.

## Verifying changes

There is no automated test or lint setup. After editing, verify by:

1. Opening `index.html` in a browser and checking the browser console for errors.
2. Playing through at least one stage, a boss fight, and a game over on the affected difficulty.
3. Checking the touch layout at a narrow viewport if the HUD or controls changed.

If you add tooling (tests, a linter, a build), document the commands here and keep them in sync with the code.

## Git

- Development branch for this work: `claude/gifted-tesla-cehdmh`. Push with `git push -u origin <branch>`.
- Write descriptive commit messages in the imperative mood, with a short summary line and a body that explains the why when it isn't obvious.
