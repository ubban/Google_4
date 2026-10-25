# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a static, single-file browser game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in `index.html`: markup, CSS in a `<style>` block, and JavaScript in one `<script>` IIFE. There is no build step, no package manager, no bundler, and no test suite.

- `index.html`: the whole application (~1,670 lines).
- `README.md`: a one-line stub.

To run the game, open `index.html` in a browser, or serve the directory with any static server.

## Architecture of `index.html`

The file has three sections, in order:

1. **`<style>`** (lines 10–359): CSS for the HUD (pill-style stats, heart lives, bomb icons), the glassmorphic overlay card, combo and stage banners, the boss health bar, touch controls, the low-health vignette, and the `.reduced-motion` class.
2. **`<body>` markup** (around lines 360–420): the `<canvas id="gameCanvas">`, HUD elements, overlay, touch controls, and a hidden `aria-live` region (`#srLive`) used for screen-reader announcements.
3. **`<script>`** (starts around line 422): one `(function () { 'use strict'; ... })();` IIFE. Nothing is exposed on `window`. Keep new code inside this IIFE unless there is a reason to change that.

Inside the script, code is grouped by `// ---------- Section ----------` comments. Follow that convention when adding code. The main sections are:

- **Config tables**: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`, `BASE_W`/`BASE_H`.
- **Persistence**: `localStorage` keys prefixed with `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
- **Adaptive performance**: `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) up or down.
- **Audio**: a Web Audio API synth (`ensureAudio`, `beep`, `noiseBurst`, `panFor`). There are no audio files. `ensureAudio` must run from a user gesture before sound plays.
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse on the canvas, a touch joystick on the left, and FIRE and BOMB buttons on the right.
- **Game logic**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `breakChain`, `addScore`, `loseLife`, `endGame`, `togglePause`.
- **Bosses**: `BOSS_PATTERNS` holds bullet-pattern generators (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` picks and runs them; `killBoss` advances the stage.
- **Rendering**: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`. Rendering is 2D canvas only.
- **Loop**: `loop(now)` runs `requestAnimationFrame`. `dt` is clamped to 50 ms so a backgrounded tab does not produce a large jump.

### Core game rules (keep in sync with code)

- Difficulty (`easy` / `normal` / `hard`) controls spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- Each stage needs `KILLS_PER_STAGE` (18) kills before a boss spawns. There are six boss types, and a theme from `THEMES` is applied per stage.
- Enemies are `grunt`, `interceptor` (weaving), and `tank` (multi-hit, from stage 2).
- Kill chains build a score multiplier (`mult`) that resets on `breakChain`.
- Weapons (Vulcan, Laser) have three power levels, raised by power-ups. Bombs clear the screen and damage bosses.

### Controls

| Action | Keyboard | Mouse | Touch |
|---|---|---|---|
| Move | Arrows / WASD | Move cursor over canvas | Drag left zone (joystick) |
| Fire | Space (hold) | Hold mouse button | Tap FIRE |
| Bomb | X or Shift | — | Tap BOMB |
| Pause | P or Esc | — | Pause button |

Keep the `#controlsHint` text in the overlay consistent with any control change.

## Conventions

- **Single file.** Do not split `index.html` into modules or add a build step unless the user asks. The no-dependency, open-and-play design is intentional.
- **No external dependencies.** Do not add CDN scripts, fonts, or packages without asking.
- **Match the local style.** The code uses `const`/`let`, arrow functions in callbacks, two-space indentation, single quotes, and short inline comments only where the logic is not obvious. Section headers use the `// ---------- Name ----------` form.
- **Keep the IIFE closed.** New state goes in the IIFE scope. Do not add globals.
- **Balance numbers live in the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`). Change tuning there rather than in inline literals.
- **Accessibility.** Preserve the `prefers-reduced-motion` handling (`reducedMotion` flag and `.reduced-motion` class), the `aria-live` announcements via `announce()`, and the keyboard-operable overlay buttons.
- **Persistence keys** use the `skyfighter.` prefix. Keep reads and writes wrapped so the game still runs if storage is unavailable, as it does now with the existing `localStorage` calls.

## Working on this repo

- **No tests or linters exist.** Verify changes by opening `index.html` in a browser (or a headless Chromium) and playing through: start, move, fire, bomb, pause, lose a life, reach a boss, game over, and check the best-score update. Also check the touch layout at phone width.
- **Quick syntax check** for the script block, if you want one without a browser:
  ```bash
  node -e "const s=require('fs').readFileSync('index.html','utf8');const m=s.match(/<script>([\s\S]*?)<\/script>/);new Function(m[1]);console.log('ok')"
  ```
- **Git.** Development happens on the designated feature branch. Commit with clear messages and push with `git push -u origin <branch>`. Do not open a pull request unless asked.
- **Scope.** Keep changes minimal and local to the feature being changed. Do not reformat unrelated parts of `index.html`, since the file is one long diff-hostile block.

## When updating this file

Update this CLAUDE.md when the structure changes: new config tables, new sections in the script, new controls, new storage keys, or a new file in the repo. Keep it factual and short, and reflect what the code does rather than plans.
