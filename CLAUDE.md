# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project. The only game code is **Sky Fighter**, a Raiden-style
vertical shoot-'em-up, contained entirely in `index.html`. `README.md` is a
placeholder with no setup instructions.

There is no build step, package manager, test suite, linter, or CI config.

## Layout

- `index.html` (~1,670 lines): the whole game.
  - Lines ~1–360: `<style>` block (HUD, overlay, touch controls, theming, reduced-motion styles).
  - Lines ~360–421: DOM markup (canvas `#gameCanvas`, overlay, HUD, touch controls, a
    polite live region `#srLive` for screen-reader announcements).
  - Lines ~422–1668: one `<script>` IIFE in strict mode containing all game logic.
- `README.md`: placeholder.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server.
Nothing needs to be installed.

## Game architecture (inside the script IIFE)

- **Fixed logical resolution**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the
  canvas to the viewport. Game coordinates should stay in base units.
- **Main loop**: `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 0.05 s, and calls
  `update(dt)` then `draw()`, only while `running && !paused`.
- **Adaptive quality**: `trackPerf()` averages the last 45 frame times and steps `quality`
  (0 full / 1 medium / 2 low) down when the frame rate drops below ~40 fps and back up above ~56 fps.
  Visual effects should respect this value.
- **Difficulty**: `DIFFICULTIES` holds per-level tuning (spawn rate, enemy speed, shoot chance,
  lives, bombs, boss HP multiplier). Change balance there, not in the logic.
- **Content tables**: `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled per
  stage), `THEMES` (six sky palettes), `BOSS_PATTERNS` (bullet patterns). Add new content by
  extending these tables and the matching spawn/draw code.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss fight.
- **Power-ups**: `life`, `bomb`, and `weapon` drops, applied in `applyPowerUp()`.
- **Drawing**: all rendering is on a single 2D canvas context in `draw()`, with helpers such as
  `drawPlane()`, `drawBossShip()`, and `drawClouds()`.
- **Audio**: Web Audio API synthesis only (`beep()`, `noiseBurst()`). There are no audio
  files. Audio is created lazily in `ensureAudio()` after a user gesture.

## Input

- Keyboard: Arrows or WASD to move (state tracked in `keys[e.code]`), Space to fire,
  `X` or Shift to bomb, `P` or Escape to pause.
- Mouse: pointer move and press on the canvas.
- Touch: virtual joystick (`#moveZone`), fire and bomb buttons, shown only on touch devices.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every access can throw in private windows or when storage is blocked. Keep new
reads and writes defensive, or at least consistent with the existing code.

## Accessibility

- Respect `prefers-reduced-motion`. The user can override it with the motion toggle, which is
  persisted. Screen shake and flashes should be gated on `reducedMotion`.
- Announce important state changes through `announce()`, which writes to `#srLive`.
- Interactive controls are real `<button>`s with labels. Keep them that way.

## Conventions

- Keep the game in a single self-contained file unless there is a clear reason to split it.
  The project has no bundler, so there is no module system to rely on.
- No external dependencies or CDN scripts. Everything is inline.
- Match the existing style: 2-space indent, single quotes in JS, `const` / `let`,
  section banners written as `// ---------- Name ----------`.
- Use `dt`-scaled movement (`speed * dt`) so gameplay stays frame-rate independent.
- Gameplay tuning constants live in the tables near the top of the script, not inline in logic.

## Verifying changes

There are no automated tests. For a change to game logic or rendering:

1. Open `index.html` in a browser (or serve it locally) and check the console for errors.
2. Play through a start, a stage clear and boss fight, a game over, and a restart.
3. Check the touch layout, or use browser device emulation, when touching input or HUD code.
4. Check that reduced-motion and mute still behave when relevant code is touched.

## Git

- Develop and push on the branch assigned to the session.
- Write commit messages that describe the behavior change, not the file edited.
