# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is one self-contained file, `index.html` (~1,670 lines): HTML, a `<style>` block, and one inline `<script>` IIFE that uses a 2D canvas. `README.md` is only a stub.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Do not add any of these unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static file server.
- There are no automated tests. Verify changes by playing the game in a browser, covering at least:
  - desktop controls (keyboard, mouse steering, click to fire)
  - touch controls (joystick, FIRE, BOMB)
  - pause and resume
  - stage transitions and boss fights
  - game over and the best-score update

## Code structure (inside `index.html`)

- **Layout and styling** (`<style>`): the canvas is a fixed logical size of 480×720 (`BASE_W`/`BASE_H`), scaled by `resize()` to fit the window, capped at 1.6×. HUD elements (`#hud`, `#hearts`, `#bombs`, `#weaponStat`, `#bossBarWrap`), overlays, and touch controls are absolutely positioned on top of the canvas.
- **Config constants** near the top of the script:
  - `DIFFICULTIES`: easy / normal / hard, controlling spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE`: kills needed to advance a stage.
  - `THEMES`: background sky palettes, cycled per stage.
  - `ENEMY_TYPES`: grunt, interceptor, tank.
  - `BOSSES`: six bosses, one per stage cycle, each with its own colors and HP multiplier.
- **Persistence**: `localStorage` keys, all prefixed `skyfighter.`:
  - `skyfighter.bestScore`
  - `skyfighter.muted`
  - `skyfighter.difficulty`
  - `skyfighter.reducedMotion`
  
  Reads and writes are not wrapped in try/catch today. Keep that in mind if you change storage access, since private or blocked-storage browsers can throw.
- **Audio**: Web Audio API, created lazily by `ensureAudio()` on first user gesture. Sound effects are built from `beep()` and `noiseBurst()` through the `sfx` table. Respect the mute flag.
- **Input**: `keys` map keyed by `e.code`; `firing` flag for mouse and touch; joystick handled by `joystickStart`/`joystickMove`/`joystickEnd`.
- **Game loop**: `update(dt)` runs game logic, `draw()` renders, and `loop(now)` drives both through `requestAnimationFrame`. `trackPerf()` adjusts a `quality` level (0 full, 1 medium, 2 low) when frame times slip.
- **Weapons and power-ups**: `player.weapon` is `'vulcan'` or `'laser'`, with `player.level` and a `weapon` power-up type that toggles between them. `fireWeapon()` builds the shots.
- **Bosses**: `spawnBoss()`, `updateBoss(dt)`, and a family of attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **Accessibility**: `announce()` writes to a live region (`#srLive`). Reduced motion (`reducedMotion`) gates screen shake, flash, and pulse effects. It defaults to the OS `prefers-reduced-motion` setting until the user toggles it.

## Controls (for reference when testing)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a smart bomb, P or Esc to pause.
- Mouse: move to steer, click and hold to fire.
- Touch: drag in the left zone to fly, tap FIRE, tap BOMB.

## Conventions

- Keep the game in a single file unless the user asks for a split. Keep new code in the existing style: vanilla JS, 2-space indent, `const`/`let`, no dependencies.
- Keep the layout working at phone width. The page sets `touch-action: none` and `overflow: hidden`, so avoid changes that add page scroll.
- Do not hard-code new storage keys outside the `skyfighter.` prefix.
- Preserve the mute and reduced-motion behavior when adding new sound or visual effects.
- When changing difficulty or balance numbers, update them in the config tables rather than inline.

## Git

- Development branch: `claude/gifted-tesla-q22gkt`. Commit and push there. Do not push to other branches without permission.
- Do not open a pull request unless the user asks for one.
