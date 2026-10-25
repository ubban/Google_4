# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a single-page, Raiden-style vertical shoot-'em-up built on HTML5 canvas. There is no build step, no package manager, no framework, and no test suite. Everything (markup, CSS, and JavaScript) lives in `index.html`. `README.md` is a placeholder.

## Repository layout

- `index.html` (~1670 lines), in three sections:
  - `<style>` (lines ~10–362): plain CSS for the HUD (`#hud`, `.stat`, `#hearts`, `#bombs`), overlay card, boss bar, touch controls, vignette/flash effects, and keyframe animations. The page sets `color-scheme: dark`.
  - `<body>` markup (lines ~363–421): `#gameCanvas`, the HUD, the boss bar, combo and stage banners, the top-bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), a screen-reader live region (`#srLive`), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
  - One `<script>` (lines ~422–1668) wrapping all logic in a single IIFE. Nothing is exported to `window`.
- `README.md`: placeholder only.

## Architecture of the script

The IIFE is organized by `// ----------` section comments. Follow that convention when adding code.

- **Config tables** (near the top of the IIFE): `DIFFICULTIES` (easy/normal/hard tuning: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (sky palettes cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank, with `minStage` gating), and `BOSSES` (six bosses with colors and `hpMul`).
- **Persistence**: `localStorage` keys are `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Reads and writes are unguarded, so keep them simple.
- **Audio**: a WebAudio synth (`beep`, `noiseBurst`, and the `sfx` object). There are no audio files. `ensureAudio()` must run from a user gesture.
- **Canvas sizing**: the logical resolution is fixed at `BASE_W=480 × BASE_H=720`. `resize()` scales the CSS size to fit the window (max scale 1.6). Game coordinates never depend on the window size.
- **Input**: keyboard (`keys` map keyed by `e.code`), touch joystick and buttons, and mouse steering (the plane glides toward the pointer; keyboard and joystick take priority).
- **Game state and logic**: `resetGame()`, `update(dt)` (movement, firing, collisions, spawning), `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`, and `killBoss`.
- **Boss attacks**: a series of `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing` helpers, selected per boss inside `updateBoss`.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. `draw()` is called only while the game is running and not paused.
- **Main loop**: `loop(now)` clamps `dt` to 0.05 s, runs `trackPerf` for adaptive quality (`quality` 0–2), and calls `update` and `draw`.

## Controls (also shown in `#controlsHint`)

- Mouse: move to steer, click to fire.
- Arrows or WASD: move. Space: fire.
- X or Shift: smart bomb. P or Esc: pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

## Working conventions

- Keep the single-file structure. Do not split out modules, add a bundler, or add dependencies unless the user asks.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, arrow callbacks for listeners, and short section-divider comments.
- Keep all gameplay numbers in the config tables or near the code that uses them, not scattered as magic values in new code.
- Use `announce()` for any new player-facing status change so screen readers get it.
- Respect the reduced-motion setting (`reducedMotion`, toggled by `#motionBtn`). `screenShake` scales magnitude and duration down, `flashHit`/`flashBomb` return early, and the `body.reduced-motion` class disables CSS pulses. New shake, flash, or pulse effects should follow the same pattern.
- Preserve touch support. Any new control needs a touch path, and `#touchControls` is only shown on touch devices.
- Do not add external assets or CDN scripts. The game is self-contained.

## Verifying changes

There are no automated tests, linters, or build commands. To verify:

1. Open `index.html` in a desktop browser and check that the game starts from the overlay, the HUD updates, and pause/mute/reduced-motion toggles work.
2. Test the affected path. For example, for boss changes, reach the relevant stage (or temporarily lower `KILLS_PER_STAGE` locally, then revert). For input changes, test both keyboard and touch (browser devtools device mode is sufficient).
3. Check the layout at phone width. The HUD and overlay must fit without horizontal scroll.
4. Run `node --check` on an extracted copy of the script if you want a syntax check. Do not commit the extracted file.

## Git

- Development branch for this repo: `claude/gifted-tesla-0u8vcn`. Push there with `git push -u origin <branch>`.
- Commit messages: a short imperative subject line (for example, `Add Sky Fighter: ...`), then a bulleted body describing what changed and why.
- Do not open pull requests unless the user asks.
