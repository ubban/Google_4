# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a tiny static web project containing one game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- `index.html` (~1,670 lines) is the entire game: inline CSS (`<style>`, roughly lines 10–361), HTML markup and HUD/overlay/touch controls (roughly lines 362–421), and one inline `<script>` IIFE with all game logic (roughly lines 422–1668), drawn on a `<canvas id="gameCanvas">`.
- `README.md` is a placeholder title only.
- There is no build step, package manager, bundler, framework, test suite, or linter. Nothing to install.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and load it.
- There are no automated tests. Verify changes by playing the game in a browser: start a run, check movement and firing, bombs, pause, boss fights, stage transitions, game over, and the difficulty buttons.
- The game uses `requestAnimationFrame` and a fixed base resolution (`BASE_W = 480`, `BASE_H = 720`) scaled by `resize()`. Check both desktop and a narrow/phone-sized viewport, since touch controls appear on touch devices.

## Architecture (inside the `<script>` IIFE)

Everything is in one closure; nothing is exported to `window`. Key pieces, in file order:

- **Constants**: `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE = 18`, `THEMES` (six sky palettes cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (six bosses, cycled by stage).
- **Persistence**: `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. All reads/writes are direct calls, so keep them inside the existing pattern and tolerate missing values.
- **Audio**: Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, `sfx`). No audio files. Audio is created lazily on the first user gesture.
- **Adaptive quality**: `trackPerf` watches frame times and steps `quality` (0 full, 1 medium, 2 low) to reduce effects when frame rate drops.
- **Main loop**: `loop(now)` clamps `dt` to 0.05 s, then calls `update(dt)` and `draw()` only when `running && !paused`.
- **Update**: `update(dt)` handles player movement, firing, enemy and bullet motion, collisions (`rectsOverlap`), power-ups, scoring/combos, stage progression, and boss updates via `updateBoss`.
- **Weapons**: `fireWeapon()` branches on `player.weapon` (`vulcan` and others) and on `player.level`.
- **Bosses**: `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `spawnBoss`, `updateBoss`, `killBoss` manage the boss lifecycle and the boss HP bar.
- **Drawing**: `draw()` renders the background (`drawClouds`), planes (`drawPlane`), boss ships (`drawBossShip`), bullets, explosions, and HUD effects. Canvas only; the HUD text lives in DOM elements.
- **Game state transitions**: `resetGame`, `loseLife`, `endGame`, `togglePause`, `showStageBanner`.
- **Accessibility**: `announce()` writes to the `#srLive` live region. `reducedMotion` (user setting, defaulting to `prefers-reduced-motion`) disables screen shake, flash, and pulse effects via a `reduced-motion` body class.

## Controls

- Move: arrow keys or WASD. Touch: on-screen joystick (`#moveZone`). Mouse: the plane glides toward the pointer while it hovers the canvas.
- Fire: Space (held), the FIRE button (touch or mouse), or mouse-down on the canvas.
- Bomb: `X` or Shift, or the BOMB button.
- Pause: `P` or `Escape`, or the pause button.
- Mute and reduced-motion toggles are in the top bar.

## Conventions

- Keep the single-file layout. Do not split into modules or add a build step unless the user asks.
- Keep the code in the existing style: `const`/`let`, 2-space indentation, `'use strict'` inside the IIFE, and small named helper functions.
- Preserve the `skyfighter.` localStorage key prefix; changing a key silently resets players' saved best score or settings.
- Any new user-visible state change should also call `announce()` if it matters to screen-reader users.
- The page currently has no external requests (the favicon is an inline `data:` URI). Keep it that way unless the user asks for a dependency.
- The canvas is a fixed 480×720 logical size; `resize()` scales the displayed size to fit the window (capped at 1.6×). Keep game coordinates in `BASE_W`/`BASE_H` space.

## Git

- Develop on the branch the session specifies (currently `claude/gifted-tesla-ethydg`). Do not push to other branches without permission.
- Do not open a pull request unless the user explicitly asks.
