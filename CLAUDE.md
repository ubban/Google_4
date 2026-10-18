# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (inline CSS, inline JavaScript, drawn on a `<canvas>`). `README.md` is a placeholder and carries no project documentation.

There is no build step, no package manager, no bundler, and no test suite. Do not add any of these unless the user asks.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no dependencies to install.

## Verifying changes

There are no automated tests or linters configured. To check a change:

1. Load `index.html` in a browser and confirm there are no console errors.
2. Play through the affected path: start a game, fire, use a bomb, pick up a power-up, and reach a boss if the change touches stages or bosses.
3. Check the change at the phone-width and desktop-width layouts, since the touch controls appear on small screens.
4. For a syntax-only check of the inline script, extract the `<script>` block and run `node --check` on it.

## Code layout (`index.html`)

The file is organized in this order:

- **`<style>`** (around lines 10–360): layout, HUD, overlay, touch controls, and the `reduced-motion` class.
- **`<body>`** (around lines 361–421): the canvas, HUD elements, boss bar, top-bar buttons, the screen-reader live region, touch controls, and the start/pause overlay with difficulty buttons and the controls hint.
- **`<script>`** (around lines 422–1668): one IIFE with `'use strict'`. Its sections, in order:
  - DOM lookups and `STORAGE_*` keys
  - Adaptive quality (`trackPerf`, `quality`)
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
  - Audio (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object)
  - HUD helpers and effects (hearts, bombs, flashes, combo pop, stage banner, screen shake)
  - Canvas sizing (`BASE_W = 480`, `BASE_H = 720`, `resize`)
  - Input: keyboard, mouse, touch joystick, fire and bomb buttons
  - Game state and lifecycle: `resetGame`, `update`, `endGame`, `togglePause`
  - Drawing: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`
  - Spawning and combat: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `updateBoss`, `killBoss`, `loseLife`
  - Boss attack patterns: `BOSS_PATTERNS` and the `fire*` helpers
  - Main loop: `loop(now)` drives `update` then `draw` through `requestAnimationFrame`, with `dt` capped at 0.05 s

## Game rules and tuning

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting lives and bombs, and a boss HP multiplier.
- **Stages**: each stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss. Defeating the boss increments `stage`. Background theme cycles through `THEMES` by `(stage - 1) % THEMES.length`.
- **Enemies** (`ENEMY_TYPES`): `grunt` (stage 1+), `interceptor` (stage 1+, weaves), and `tank` (stage 2+, 4 HP).
- **Bosses** (`BOSSES`): six bosses, cycled in order. Each has an `hpMul` and a color scheme. Attack patterns are in `BOSS_PATTERNS`.
- **Power-ups**: `life`, `bomb`, and `weapon`, applied in `applyPowerUp`. Weapon power-ups raise the level from 1 to 3, then swap between `vulcan` and `laser` and reset to level 1.
- **Scoring**: `addScore` with a combo multiplier, reset by `breakChain`. Killing a boss awards 500.
- **Lives and bombs** are capped by `maxLives` and `maxBombs`.

Keep gameplay numbers in the existing tables. Do not scatter new magic numbers through the update loop.

## Persistence

The game uses `localStorage` with the keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Every read and write is currently unguarded. If you add storage access, follow the same key prefix and keep values as strings. Note that the `reducedMotion` default comes from `prefers-reduced-motion` when no saved value exists.

## Accessibility and input

- Screen-reader announcements go through `announce()`, which writes to the `#srLive` live region. Use it for important state changes such as game over, a new stage, or a boss defeat.
- Keyboard controls: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Escape to pause.
- Mouse controls: move to steer and click to fire. Touch controls: a left-side joystick, plus FIRE and BOMB buttons.
- The reduced-motion toggle suppresses screen shake, flashes, and pulses. Respect the `reduced-motion` body class for any new visual effect that moves the screen or flashes.

## Conventions

- Plain HTML, CSS, and JavaScript only. Do not introduce frameworks, CDNs, or build tooling.
- Keep everything in `index.html` unless the user asks to split it.
- Match the existing style: two-space indentation, single quotes in JavaScript, `const` and `let` with arrow functions where they already appear, and short section comments using the `// ---------- Name ----------` banner form.
- Keep the game loop allocation-light. Reuse existing arrays and objects where practical, since the adaptive quality system already reacts to slow frames.

## Git

- Develop on the branch the session specifies. The current development branch is `claude/gifted-tesla-h56cm4`.
- Commit with clear messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
