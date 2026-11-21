# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small, dependency-free browser game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up). The entire game lives in one file:

- `index.html` (~1,670 lines): inline `<style>`, the HUD/overlay markup in `<body>`, and one inline `<script>` wrapped in an IIFE.
- `README.md`: a placeholder title only. It has no setup or usage docs.

There is no `package.json`, no build step, no bundler, no test suite, and no linter configuration. Assets are not loaded from disk or the network: all sound is synthesized with the WebAudio API, and all graphics are drawn on a `<canvas>`.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server` from the repo root. No install step is needed.

Verification is manual. There are no automated tests, so after a gameplay change, load the page in a browser and play through the affected path. Check the desktop and touch controls if the change touches input.

## Layout of `index.html`

| Section | Contents |
| --- | --- |
| `<style>` | Layout, HUD, overlay card, touch controls, and `body.reduced-motion` overrides. |
| `<body>` | `#container` holds the `<canvas id="gameCanvas">`, the vignette and flash overlays, the HUD (`#hud`), boss bar, combo/stage banners, top buttons (mute, reduced motion, pause), touch controls, and the start/title overlay (`#overlay`). |
| `<script>` | One IIFE with `'use strict'`. Sections are marked by `// ---------- Name ----------` comments. |

Sections inside the script, in order of appearance:

1. **DOM refs and constants**: `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
2. **Adaptive performance**: `trackPerf` samples frame time and steps a `quality` level (0 full, 1 medium, 2 low) to reduce effects on slow devices.
3. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is created lazily on the first user gesture.
4. **Feedback helpers**: hearts/bombs rendering, hit and bomb flashes, combo pops, stage banner, screen shake.
5. **Settings**: difficulty selection, reduced-motion toggle, mute toggle.
6. **Canvas sizing**: `resize()` keeps a fixed logical resolution of `480x720` (`BASE_W` / `BASE_H`) and scales the CSS size to fit the window, capped at 1.6x.
7. **Input**: keyboard (`keys` map keyed by `e.code`), virtual joystick for touch, mouse targeting via `mouseTarget`.
8. **Game state and entities**: `resetGame`, drawing functions (`drawPlane`, `drawBossShip`), spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), collisions (`rectsOverlap`), scoring and chains (`addScore`, `breakChain`), `useBomb`, `fireWeapon`.
9. **Update and boss patterns**: `update(dt)` is the main simulation step. `BOSS_PATTERNS` and the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring) implement boss attacks. `updateBoss`, `killBoss`, and `loseLife` handle boss and player-death transitions. `endGame` and `togglePause` manage the run lifecycle.
10. **Rendering**: `drawClouds`, then `draw()`, which renders one frame.
11. **Main loop**: `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then runs `update` and `draw` while the game is running and not paused. A `startBtn` click handler starts or resumes the game.

## Game rules and constants

Change these values with care. Several are coupled, and gameplay tuning is the most likely source of unintended effects.

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy shoot chance, starting `lives`, starting `bombs`, and a boss HP multiplier.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss. The boss for stage *n* is `BOSSES[(stage - 1) % BOSSES.length]`, so the six bosses cycle. The background `THEMES` also rotate per stage.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2).
- **Power-ups** (`applyPowerUp`): `life` (capped at `maxLives`), `bomb` (capped at `maxBombs`), and `weapon`. The weapon power-up raises the level to 3, then toggles between `vulcan` and `laser` and resets to level 1.
- **Controls**: mouse moves and clicks to fire. Arrows or WASD move, Space fires, X or Shift uses a bomb, and P or Esc pauses. On touch, a left-side drag steers, FIRE fires, and BOMB bombs. The key codes are handled in the `keydown` listener and the `keys` map.

## Persistence

Settings and the best score are stored in `localStorage` under these keys. Keep them stable, because changing a key resets the player's saved data:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reduced motion defaults to the `prefers-reduced-motion` media query until the player makes an explicit choice. It disables screen shake, flashes, and pulses.

## Conventions

- **Keep it a single file.** Do not add a build step, package manager, or external script or stylesheet dependencies unless the user explicitly asks for it. The project is deliberately self-contained, and the audio and graphics are generated in code.
- **Keep the IIFE and `'use strict'`.** Helpers stay private to the IIFE. Do not add globals.
- **Use the existing section comments** (`// ---------- Name ----------`) when adding a new subsystem, so the file stays navigable.
- **Stay frame-rate independent.** Movement and timers use `dt` in seconds. Do not hard-code per-frame increments.
- **Respect the reduced-motion and quality paths.** New screen shake, flashes, or particle-heavy effects should check `reducedMotion` and the `quality` level, following the existing helpers.
- **Keep the logical resolution fixed.** Coordinates are in the 480x720 canvas space. Scaling is handled by CSS in `resize()`, so do not change the canvas backing size per device.
- **Accessibility:** announce important state changes through `announce()`, which writes to the `#srLive` live region. Keep buttons labeled, and keep the `title` attributes accurate when toggling states.
- **Touch and desktop parity:** if you add or change an action, wire it to both the keyboard/mouse path and the touch controls.
- **Git:** the working branch for this repo is `claude/gifted-tesla-5f9pzt`. Do not push to other branches without explicit permission.

## Making changes

1. Read the affected section first. Most game logic lives in `update(dt)` and `draw()`, and many features touch both.
2. Make the smallest change that works, and avoid restructuring unrelated code. The file has no tests, so reasoning about state transitions carefully matters.
3. Verify in a browser: start a run, exercise the change, and confirm that pausing, game over, and restart still work. Check the console for errors.
4. Keep the existing comments and code style (2-space indentation, single quotes in JavaScript, and short helper functions).
