# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

- `index.html` (~1,670 lines): the entire game. CSS is in a `<style>` block (~lines 10–362), markup and the canvas (`#gameCanvas`) follow, and all game logic is in one `<script>` IIFE (starts ~line 422).
- `README.md`: a placeholder. It holds no setup or usage instructions.

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Open `index.html` in a browser to run the game. No install is needed.

## Repository layout

```
.
├── index.html   # The game: markup, styles, and script in one file
├── README.md    # Placeholder
└── CLAUDE.md    # This file
```

Keep the game as a single self-contained file unless the user asks for a split. Do not add external dependencies, CDN scripts, or a build system without an explicit request.

## Code structure (inside `index.html`)

The script is one IIFE with `'use strict'`. Its sections, in order:

1. **DOM references and constants.** `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, and `BOSSES`.
2. **Persistence.** Best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under the `skyfighter.*` keys.
3. **Adaptive performance.** `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low).
4. **Audio.** Web Audio via `ensureAudio`, `beep`, `noiseBurst`, and a named `sfx` table. Audio starts only after a user gesture.
5. **HUD and effects.** `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, and `screenShake`.
6. **Input.** Keyboard (`keys` map), touch joystick and fire/bomb buttons, and mouse on the canvas.
7. **Game state.** `resetGame`, `update(dt)`, `draw()`, and the `loop(now)` rAF loop. Gameplay logic lives in `update` and rendering in `draw`. Keep that split.
8. **Weapons, power-ups, and bosses.** `fireWeapon`, `applyPowerUp`, the `fire*` pattern helpers, `BOSS_PATTERNS`, `updateBoss`, and `killBoss`.
9. **Lifecycle.** `loseLife`, `endGame`, `togglePause`, and the start button handler.

Use `announce()` for screen-reader updates when game state changes (game over, stage changes, and similar).

## Conventions

- **Plain JavaScript and CSS.** Target modern browsers. Use `const` and `let`, arrow functions, and template strings to match the existing style.
- **Timing uses `dt`.** Movement and timers are scaled by the frame delta. Do not hardcode per-frame step sizes.
- **Respect reduced motion.** Effects like screen shake and hit flash check the `reducedMotion` flag. Any new visual shake or flash should do the same.
- **Mobile and desktop.** Changes to controls must keep both touch (`touchControls`) and keyboard/mouse input working. The layout scales to a `480×720` base via `resize()`.
- **Persist with the existing keys.** Store new settings under `skyfighter.<name>` in `localStorage`, following the existing pattern.
- **Keep it tunable.** Balance numbers (spawn rates, lives, boss HP multipliers) belong in the constant tables at the top of the script, not scattered through the logic.

## Working on the game

- **Verify changes in a browser.** No automated tests exist. Load `index.html` in a browser and play through the affected path: start, fire, bomb, pause, game over, and a stage change, as relevant. Check the browser console for errors.
- **Check both input modes** when touching controls, and both reduced-motion states when touching effects.
- **Check the layout at phone width** when touching CSS or the HUD.
- **Do not reformat the whole file.** Keep diffs focused so the change is easy to review.

## Git

- Development branch for this work: `claude/gifted-tesla-vuj7va`. Commit and push there.
- Do not open pull requests unless the user explicitly asks.
- Commit messages describe the change in the imperative mood, with a short summary line followed by a body when the change needs explaining.
