# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire app lives in one file, `index.html`, which contains inline CSS (`<style>`, roughly lines 10–360), the markup for the HUD and overlays (roughly lines 361–421), and one inline `<script>` (roughly lines 422 onward) wrapped in an IIFE with `'use strict'`.

`README.md` is a placeholder and contains only the project title.

There is no build step, package manager, bundler, test suite, or linter. Nothing needs to be installed. To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Repository layout

```
index.html   # The whole game: CSS, HTML, and JS
README.md    # Placeholder
CLAUDE.md    # This file
```

## Architecture of `index.html`

The script is organized as a set of sections, each marked with a `// ---------- Name ----------` comment. Keep new code inside the matching section.

- **DOM references and storage keys.** Element lookups are `const`s at the top. `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
- **Adaptive performance.** `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) up or down to keep the frame rate stable.
- **Configuration tables.** These are data-driven and are the first place to look when tuning gameplay:
  - `DIFFICULTIES`: `easy`, `normal`, `hard`, with spawn timing, enemy speed, enemy fire chance, starting lives and bombs, and a boss HP multiplier.
  - `KILLS_PER_STAGE`: kills needed to advance a stage (18).
  - `THEMES`: six sky palettes that cycle with stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, minimum stage 2).
  - `BOSSES`: six bosses in order (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with colors and an `hpMul`.
  - `BOSS_PATTERNS`: bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Audio.** Web Audio synthesis only, with no audio files. `ensureAudio` creates the context lazily on the first user gesture. Helpers are `beep` and `noiseBurst`, with named sound effects in `sfx`, and `EXPLODE_SFX` maps enemy types to explosion sounds. The mute state persists.
- **Input.** Keyboard, mouse, and touch are all handled.
  - Keyboard: arrows or WASD move, Space fires, X or Shift bombs, P or Esc pauses.
  - Mouse: move to steer, click to fire.
  - Touch: a joystick on the left (`moveZone`) plus FIRE and BOMB buttons.
  - Keyboard state is tracked in `keys[e.code]`.
- **Game state and update loop.** `resetGame` initializes state. `update(dt)` advances the simulation, and `draw()` renders it. `loop(now)` drives both with `requestAnimationFrame`. Pause is handled by `togglePause`.
- **Weapons and power-ups.** `fireWeapon` handles two weapon types, `vulcan` and `laser`, each with three levels. `applyPowerUp` handles three drop types: `life`, `bomb`, and `weapon`. Picking up a `weapon` power-up upgrades the level; past level 3 it switches weapons.
- **Bosses.** `spawnBoss`, `updateBoss`, and `killBoss` manage the boss fight, including the health bar in the boss HUD.
- **Game flow.** `loseLife` and `endGame` handle death and game over. `flashHit`, `screenShake`, and `showStageBanner` provide feedback.

## Conventions

- **Single file.** Keep CSS, markup, and JS in `index.html` unless the user asks for a split. Do not introduce a build step or framework without being asked.
- **No dependencies.** Use plain browser APIs (Canvas 2D, Web Audio, `localStorage`, `matchMedia`). Do not add CDN scripts or npm packages.
- **Match the local style.** Use 2-space indentation, single quotes in JS, `const` by default, and the existing section-comment banners. Name new constants in `UPPER_SNAKE_CASE` when they are configuration (like `KILLS_PER_STAGE`) and use camelCase for functions and variables.
- **Gameplay tuning goes in the config tables.** Prefer adding or changing values in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES` over hard-coding numbers in the update logic.
- **Persist player settings through the `skyfighter.*` keys.** Wrap new `localStorage` access in the same pattern the file already uses, since storage can be unavailable in some browsers.
- **Keep it responsive.** The layout must work on mobile. Touch controls are shown for touch devices, and the canvas scales in `resize()` with a base size of 480×720.
- **Respect reduced motion.** Effects should honor the `reducedMotion` flag, which defaults from the `prefers-reduced-motion` media query and can be toggled in the UI. `screenShake` scales down when it is set, and `body.reduced-motion` disables several CSS animations.
- **Accessibility.** Announcements for screen readers go through `announce()` using the `srLive` live region.

## Verifying changes

There are no automated tests or linters. To check a change:

1. Open `index.html` in a browser, or serve the directory with a static server.
2. Start a game and exercise the area you changed on desktop and in a mobile-sized viewport.
3. Check the browser console for errors. A clean console is the main signal available.
4. For changes to gameplay numbers, play through at least one stage boss on each affected difficulty.

If you add a test setup later, document the commands here.

## Git workflow

- Develop on the branch the session specifies. The current working branch is `claude/gifted-tesla-9y7xug`.
- Write clear, descriptive commit messages that explain why the change was made.
- Do not open a pull request unless the user asks for one.
