# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 canvas. The whole game is one self-contained file:

- `index.html` (~1,670 lines): inline `<style>`, markup, and one inline `<script>` IIFE. There are no external assets, dependencies, or frameworks.
- `README.md`: minimal placeholder.

There is no package manager, build step, bundler, linter, or test suite. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: HTML overlay UI, CSS, and JavaScript |
| `README.md` | Project readme |

## Architecture of `index.html`

The script is wrapped in `(function () { 'use strict'; ... })()`, so nothing leaks to `window`. Keep new code inside that IIFE.

Major sections, in the order they appear in the script:

1. **DOM references**: all `document.getElementById` lookups are cached as `const`s at the top.
2. **Constants**: `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
3. **Persistence**: best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under the `skyfighter.*` prefix.
4. **Adaptive quality**: `trackPerf` samples frame times and steps `quality` (0 full, 1 medium, 2 low) to keep the frame rate up.
5. **Audio**: a lazily created `AudioContext` (`ensureAudio`) and a synthesized `sfx` object built from `beep` and `noiseBurst`. No audio files are loaded.
6. **HUD helpers**: hearts, bombs, combo pop, stage banner, screen shake, flash, and screen-reader announcements via `announce` (the `#srLive` region).
7. **Layout**: `resize` scales the fixed `480×720` logical canvas to fit the window, capped at 1.6×.
8. **Input**: keyboard state in `keys` (keyed by `KeyboardEvent.code`), mouse/pointer steering, a touch joystick, and FIRE/BOMB touch buttons.
9. **Game state**: `resetGame`, `update(dt)` (movement, firing, collisions, spawning), `applyPowerUp`, `useBomb`, `fireWeapon`.
10. **Bosses**: `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, and others). `updateBoss` and `killBoss` drive the boss fight.
11. **Rendering**: `draw()` renders each frame, with `drawPlane`, `drawBossShip`, and `drawClouds`.
12. **Main loop**: `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05 s, and runs `update` and `draw` only while `running && !paused`.

### Controls (keep the hint in sync)

The in-game help text lives in `#controlsHint` in the markup. If you change a binding, update that text too.

- Mouse: move to steer, click to fire
- Arrows / WASD: move
- Space: fire
- X or Shift: bomb
- P or Esc: pause
- Touch: drag left side to fly, FIRE and BOMB buttons on the right

## Conventions

- **Single file.** Do not split the game into multiple files, add a bundler, or add dependencies unless the user asks for it.
- **No external network requests.** The game is fully self-contained. Do not add CDN scripts or remote fonts without asking.
- **Match the existing style.** Use `const`/`let`, arrow functions for callbacks, two-space indentation, and single quotes in JS. Section headers use the `// ---------- Name ----------` form. Add new constants to the existing tables (`ENEMY_TYPES`, `BOSSES`, `DIFFICULTIES`, `THEMES`) instead of hard-coding values inline.
- **Keep the render loop cheap.** `update` and `draw` run every frame. Avoid allocating per-frame objects where a reuse pattern already exists, and respect the `quality` level when adding visual effects.
- **Respect reduced motion.** Effects that shake or flash should check the `reducedMotion` flag. `flashHit` and `flashBomb` return early when it is set, and `screenShake` scales its intensity and duration down.
- **Accessibility.** Announce important game-state changes through `announce()` so screen readers pick them up.
- **Persist with the `skyfighter.` prefix.** Wrap new `localStorage` keys the same way the existing ones are written.

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a desktop browser and check the affected behavior by hand: start, fire, bomb, pause, boss fight, stage transition, and game over.
2. Check the mobile layout and touch controls with browser device emulation when the change touches input or HUD layout.
3. Confirm the browser console shows no errors.

If you add a test harness or tooling, document the commands here.

## Git workflow

- Develop on the branch the session specifies. Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless the user asks for one.
