# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines).
- `README.md` — minimal project title only.

There is no build system, package manager, bundler, test suite, or linter. Nothing to install.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture of `index.html`

- **Layout**: `#container` holds a `<canvas id="gameCanvas">` plus HUD (`#hud`, `#topBar`, `#bossBarWrap`), overlays (`#overlay`, `#stageBanner`, `#comboPop`), touch controls (`#touchControls`), and screen-reader live region (`#srLive`).
- **Script scope**: all game code lives inside a single IIFE with `'use strict'`. Nothing is exported to `window`; keep new code inside that closure unless there is a reason not to.
- **Rendering**: a 2D canvas drawn at a fixed logical size `BASE_W × BASE_H` (480 × 720), scaled by `resize()`.
- **Main loop**: `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 0.05 s, calls `update(dt)` then `draw()` while `running && !paused`.
- **Adaptive quality**: `trackPerf()` samples frame times and steps `quality` (0 full / 1 medium / 2 low) to keep the frame rate up.
- **Audio**: a Web Audio API synth (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object). No audio files. Audio is created lazily on the first user gesture.

### Game data (top of the script)

Tuning lives in declarative constants — prefer editing these over hard-coding numbers:

- `DIFFICULTIES` — `easy` / `normal` / `hard` settings: spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier.
- `KILLS_PER_STAGE` — kills needed to advance a stage.
- `THEMES` — background palettes cycled per stage.
- `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (multi-hit); each has size, HP, speed multiplier, score, and `minStage`.
- `BOSSES` — boss names, colors, and HP multiplier; boss attack patterns are in `BOSS_PATTERNS` and `updateBoss()`.

### Gameplay systems

- **Weapons**: `player.weapon` with levels; `fireWeapon()` implements the spread logic (e.g. `vulcan` widens at levels 2 and 3).
- **Power-ups**: types `life`, `bomb`, `weapon`; applied in `applyPowerUp()`, spawned by `maybeSpawnPowerUp()`.
- **Score / combo**: `addScore()` and `breakChain()` manage the multiplier (`#mult`).
- **Bombs**: `useBomb()` clears the screen; bomb count comes from difficulty.
- **Lives**: `loseLife()`; `endGame()` finalizes the run.
- **Pause**: `togglePause()`.

### Controls

| Input | Action |
| --- | --- |
| Arrow keys or WASD | Move (read in `update()`) |
| Mouse move over canvas | Steer (`mouseActive`) |
| Space / mouse button / touch FIRE | Fire |
| `X` or Shift | Bomb |
| `P` or Escape | Pause |
| Touch: drag in left zone (joystick), FIRE and BOMB buttons on the right | Move, fire, bomb |

### Persistence

`localStorage` keys (all prefixed `skyfighter.`):

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads and writes to `localStorage` are currently not wrapped in `try/catch`. If you touch that code, consider guarding it, since storage can be unavailable in private windows or with blocked site data.

### Accessibility

- Honors `prefers-reduced-motion` on first load, with a user toggle (`#motionBtn`) that persists.
- Screen-reader announcements go through `announce()` into `#srLive`.
- Keep new HUD or overlay text reachable via that live region where it matters.

## Conventions

- **Single file**: keep everything in `index.html` unless the user asks for a split. Don't add a build step or dependencies without a clear reason.
- **No external network dependencies**: the page should work offline. Don't add CDN links without asking.
- **Style**: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks. Match the surrounding code.
- **Comments**: section headers use `// ---------- Name ----------`. Keep comments short and explain *why*, not *what*.
- **Mobile**: the game targets phones too. Test touch changes at phone width and keep the touch controls working.

## Verifying changes

There are no automated tests. For changes:

1. Open the page in a browser (or use the `run` skill) and check that it loads without console errors.
2. Play through at least one stage on the affected difficulty, and exercise the code you changed (fire, bomb, pause, boss, game over as relevant).
3. For UI or layout changes, check both desktop and phone-sized viewports.

Quick syntax check of the script block (needs Node, no files written):

```sh
sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' | node -e "new Function(require('fs').readFileSync(0,'utf8')); console.log('syntax OK')"
```

## Git

- Develop on the branch assigned for the session; don't push to other branches without permission.
- Keep commits focused. Write messages in the imperative mood, summarizing the gameplay or UI change.
