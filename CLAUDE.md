# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single HTML5 canvas page.

- `index.html` is the entire game: markup, CSS (`<style>`, ~lines 10–359), and JavaScript (`<script>`, ~lines 422–1667).
- `README.md` is a placeholder and has no project documentation.
- There is no build step, package manager, bundler, framework, test suite, or lint config. No `package.json` exists.
- No external assets. Graphics are drawn on canvas each frame and audio is synthesized with WebAudio.

To run it, open `index.html` in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Repository layout

```
index.html   # The whole game (HTML + CSS + JS in one file)
README.md    # Placeholder
CLAUDE.md    # This file
```

Keep the game as a single self-contained file unless the user asks for a restructure. Do not add a build system or dependencies without a request.

## Architecture of `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`) with no globals. Major sections, in file order:

1. **DOM references**: `const` lookups for HUD, overlay, touch controls, and buttons.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`).
3. **Adaptive performance**: `trackPerf` samples frame times and steps `quality` (0 full, 1 medium, 2 low).
4. **Tuning tables**:
   - `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier)
   - `KILLS_PER_STAGE` (currently 18)
   - `THEMES` (one background palette per stage, cycled)
   - `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`, with `minStage` gating)
   - `BOSSES` (six boss designs, with colors and `hpMul`)
   - `BOSS_PATTERNS` (bullet-pattern functions: radial, aimed spread, columns, spiral, cross, homing, double ring)
5. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and named SFX. Audio is created lazily on the first user gesture.
6. **HUD helpers**: hearts, bombs, combo popups, stage banner, screen shake, flash, mute and motion toggles.
7. **Layout and input**: `resize()` scales a fixed `BASE_W × BASE_H` (480×720) canvas. Keyboard state is kept in `keys[]`. Mouse, touch joystick, and FIRE/BOMB buttons are wired on touch devices.
8. **Game state**: `resetGame()` initializes the `let` variables (`score`, `lives`, `bombCount`, `stage`, `kills`, `stageKills`, `bossWarned`, and others).
9. **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
10. **Simulation**: `update(dt)` handles movement, firing, spawning, collisions, and stage progression. `updateBoss(dt)`, `killBoss()`, `loseLife()`, `endGame()`, and `togglePause()` also live here.
11. **Main loop**: `loop(now)` clamps `dt` to 0.05 s, then calls `update` and `draw` when running and not paused.

### Game flow

- A stage ends after `KILLS_PER_STAGE` kills, then a boss spawns about 1.2 s later (`bossWarned` prevents repeats).
- Defeating a boss increments `stage`, which changes the theme and difficulty curve.
- Losing all lives calls `endGame()`, which updates the best score and shows the overlay.

### Controls

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag left side to fly (virtual joystick), tap FIRE and BOMB.

## Conventions

- **Style**: match the existing code. Two-space indentation, single quotes in JS, `const` by default, and `let` only for mutated state. Section headers use `// ---------- Name ----------`.
- **Balance changes** belong in the tuning tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than in scattered literals.
- **Accessibility**: keep the `reducedMotion` path working for any new shake, flash, or pulse effect. Announce important state changes through `announce()` so screen readers pick them up.
- **Persistence**: any new saved setting should use a `skyfighter.*` key and be wrapped in the same `localStorage` pattern already used. Storage can throw in some privacy modes, so consider a try/catch for new reads and writes.
- **Mobile**: anything new that needs input must work with both keyboard and touch. Keep the `touch-action: none` behavior on the game surface.
- **Audio** must be started from a user gesture (`ensureAudio()`), since browsers block autoplay.

## Verifying changes

There are no automated tests. To verify a change:

1. Open `index.html` in a browser, or serve the folder locally.
2. Check the browser console for errors on load and during play.
3. Play through at least one stage transition and one boss fight when changing combat, spawning, or boss code.
4. For layout or HUD changes, check both a desktop viewport and a phone-sized viewport (touch controls appear only on touch-capable devices).
5. Check that pause, mute, reduced-motion, and difficulty toggles still persist across reloads.

If a change cannot be exercised in a browser, say so explicitly rather than claiming it works.

## Git

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-ea0nr3`). Do not push to `main` directly.
- Write descriptive commit messages in the imperative mood that explain the gameplay or code change.
- Do not open pull requests unless asked.
