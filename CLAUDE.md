# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-file HTML5 canvas game.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (`<body>` HUD and overlays), inline CSS (`<style>`), and inline JavaScript (`<script>`). About 1,670 lines. |
| `README.md` | Placeholder title only. |

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. Everything is inline.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Architecture of `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`) with no globals.

- **Rendering:** a single `<canvas id="gameCanvas">` drawn with the 2D context. The logical play field is `BASE_W = 480` by `BASE_H = 720`, scaled to fit by `resize()`.
- **Main loop:** `requestAnimationFrame(loop)` calls `update(dt)` then `draw()` while `running && !paused`. `dt` is clamped to 50 ms.
- **Adaptive quality:** `trackPerf()` samples frame times over 45 frames and steps a `quality` level (0–2) down when the frame rate drops below about 40 fps, and up when it is above about 56 fps.
- **Difficulty:** the `DIFFICULTIES` table (`easy`, `normal`, `hard`) sets spawn rates, enemy fire chance, starting lives, bombs, and a boss HP multiplier. Selection persists via `localStorage`.
- **Stages:** `KILLS_PER_STAGE = 18`. Each stage advances the `THEMES` palette. `BOSSES` and `BOSS_PATTERNS` define the boss encounters (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Weapons and power-ups:** `fireWeapon()` implements the Vulcan/Laser system with three levels. `applyPowerUp()` handles weapon level, bomb, and extra life pickups. `useBomb()` clears the screen and damages bosses.
- **Scoring:** `addScore()` applies a kill-chain multiplier. `breakChain()` resets it.
- **Persistence:** `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write is a plain call, so guard any new storage access the same way if it might run in a private window.
- **Audio:** Web Audio API, created lazily by `ensureAudio()` on first user gesture. Mute state is in `STORAGE_MUTE`.
- **Accessibility:** `announce()` writes to the `srLive` region for screen readers. A reduced-motion toggle (`STORAGE_REDUCED`) disables screen shake and similar effects.

### Controls

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause. Space's default is suppressed.
- Mouse: move to steer, click to fire.
- Touch: drag the left zone for a virtual joystick, tap FIRE, tap BOMB.

## Conventions

- **Keep it a single file** unless the user asks to split it. The project is intentionally dependency-free and opens straight from disk.
- **Match the existing style:** 2-space indentation, `const`/`let`, arrow functions for callbacks, and named `function` declarations for game systems.
- **Use the existing naming:** `STORAGE_*` for `localStorage` keys, `SCREAMING_SNAKE_CASE` for tuning tables and constants, `fire*` for boss projectile patterns, `draw*` for rendering helpers.
- **Tuning values live in tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`). Prefer changing those over hard-coding numbers in logic.
- **Mobile and desktop both matter.** Any new input must work with keyboard, mouse, and touch, and HUD changes must hold up at phone widths.
- **Respect reduced motion** for any new screen effect. Gate it on `reducedMotion`.
- **Keep overlays and HUD in sync** with the state variables. Use the existing helpers (`renderHearts`, `renderBombs`, `showStageBanner`, `showComboPop`) rather than writing new DOM updates inline.

## Verifying changes

There are no automated tests. To check a change:

1. Load `index.html` in a browser and confirm there are no console errors.
2. Play through a full start, a few stages, a boss, a bomb, a power-up, pause/resume, and game over.
3. Check each difficulty, mute and reduced-motion toggles, and that the best score persists across a reload.
4. Test touch controls with device emulation at phone width.

A headless Chromium with Playwright is a reasonable way to automate step 1 and step 2 if needed.

## Git

- Development branch for this session: `claude/gifted-tesla-res9bx`. Do not push to other branches without explicit permission.
- Commit messages in this repo are descriptive, with a short summary line followed by a bulleted body explaining what changed and why.
- Do not open a pull request unless the user asks for one.
