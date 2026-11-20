# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game called **Sky Fighter**: a Raiden-style vertical shoot-em-up. The entire game lives in one self-contained file, `index.html`, with inline CSS and JavaScript. `README.md` is a placeholder and contains no setup instructions.

There is no build step, package manager, bundler, test suite, linter, or CI configuration in the repo.

## Repository layout

```
index.html   # The whole game: markup (lines ~1-361 head/CSS, ~361-421 body/HUD/overlay), inline <script> (~422-1667)
README.md    # Placeholder
CLAUDE.md    # This file
```

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static file server, for example:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code structure (inside `index.html`)

The script is one IIFE with `'use strict'`. Everything is private to that closure; nothing is exposed on `window`.

- **DOM refs**: all `getElementById` / `querySelector` lookups are grouped at the top of the script.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access is a plain call with no try/catch, so keep that in mind if you change storage handling.
- **Config tables**: `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `THEMES` (sky gradients, cloud and sun colors, one per stage cycle), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six boss designs). `KILLS_PER_STAGE` is 18.
- **Rendering**: a fixed logical canvas of `480 x 720` (`BASE_W`, `BASE_H`). `resize()` scales it to fit the window (capped at 1.6x) using CSS size only. Drawing helpers include `drawPlane`, `drawBossShip`, `drawClouds`, and the main `draw()`.
- **Audio**: a Web Audio API synth (`ensureAudio`, `beep`, `noiseBurst`). No audio files are loaded. Audio is created lazily on the first user gesture.
- **Input**: keyboard (`keys` map keyed by `e.code`), mouse/pointer steering and firing via `canvasPointFromEvent`, and a touch joystick plus FIRE/BOMB buttons (shown only when `isTouchDevice`).
- **Game state and logic**: `resetGame()`, `update(dt)` (the main simulation step), weapon firing (`fireWeapon`), power-ups (`maybeSpawnPowerUp`, `applyPowerUp`), scoring with a combo multiplier (`addScore`, `breakChain`), bombs (`useBomb`), and boss attack patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`).
- **Game flow**: `loseLife`, `endGame`, `togglePause`, and the `loop(now)` requestAnimationFrame loop. `dt` is clamped to 50 ms.
- **Adaptive quality**: `trackPerf` averages frame time over 45 frames and steps a `quality` level (0 full, 1 medium, 2 low) up or down.
- **Accessibility**: `announce()` writes to an `aria-live` region. A reduced-motion toggle controls screen shake, flashes, and pulses. The `prefers-reduced-motion` media query sets the default.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a split. Do not add a bundler or package manager without a clear reason.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, and arrow functions for callbacks.
- Put new tunable numbers in the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than scattering magic numbers through the update logic.
- Use the `skyfighter.` prefix for any new `localStorage` key.
- Keep the 480 x 720 logical resolution in mind when changing positions, sizes, or the HUD layout.
- Respect the reduced-motion setting for any new screen shake, flash, or pulse effect.

## Verifying changes

There is no automated test suite. Verify by:

1. **Syntax check the script.** Extract the inline `<script>` and run it through Node:
   ```sh
   f=$(mktemp --suffix=.js) && sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > "$f" && node --check "$f"; rm -f "$f"
   ```
2. **Play it in a browser.** Check the start overlay, all three difficulties, shooting, bombs, power-ups, a stage transition, a boss fight, pause, mute, reduced-motion toggle, game over, and the best-score persistence across reloads. On a touch-capable viewport, check the joystick and the FIRE/BOMB buttons.

## Git

- The development branch for this work is `claude/gifted-tesla-ftvg9w`. Commit and push there unless told otherwise.
- Do not open a pull request unless the user asks for one.
