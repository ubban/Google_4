# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single self-contained HTML file.

- `index.html` (~1,670 lines): the entire game — markup, CSS, and JavaScript in one file. No build step, no dependencies, no external assets. Audio is synthesized with WebAudio.
- `README.md`: a minimal title stub. It contains no setup or usage instructions.

There is no `package.json`, test suite, linter, or CI configuration.

## Running the game

Open the file directly in a browser, or serve the directory with any static server:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

No install step is needed.

## Code structure of `index.html`

- **Lines ~1–361 (`<style>`)**: CSS. Colors and layout are hard-coded; `color-scheme: dark`. HUD, overlay, touch controls, and vignette/flash effects are styled here.
- **Lines ~362–420 (markup)**: `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`), boss bar, stage banner, top-bar buttons (mute / reduced motion / pause), touch controls (`#touchControls`: joystick, FIRE, BOMB), and the start/title `#overlay`.
- **Lines ~422–1669 (`<script>`)**: one IIFE (`'use strict'`) containing all game logic:
  - `DIFFICULTIES` (~485): per-difficulty tuning (spawn rate, enemy speed and fire chance, lives, bombs, boss HP multiplier). `KILLS_PER_STAGE = 18` (~491).
  - Persistence (~515–540): `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access is a plain call with no try/catch, so it will throw in environments where storage is blocked.
  - Audio (~542+): `ensureAudio()` lazily creates the `AudioContext`; `beep()` and `noiseBurst()` synthesize all sound effects.
  - Input (~769–870): keyboard, joystick/touch, and mouse handlers.
  - Game state and loop: `resetGame()` (~875), `update(dt)` (~1148), `draw()` (~1520), `loop(now)` (~1640). Drawing is done with the 2D canvas API via helpers like `drawPlane` and `drawBossShip`.
  - Weapons and bosses: `fireWeapon()` (~1124), `applyPowerUp()` (~1304), and a family of boss pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) driven by `updateBoss()` (~1411).
  - Lifecycle: `loseLife()` (~1456), `endGame()` (~1469), `togglePause()` (~1491).

## Controls (for reference when changing input code)

- Move: arrow keys or WASD. Fire: Space (or hold the FIRE button). Bomb: X or Shift.
- Pause: P or Escape.
- Touch: left-side joystick, right-side FIRE and BOMB buttons.
- Mouse: move pointer over canvas to steer; mouse down fires.

## Conventions

- Keep the game in a single file unless the user asks for a split. The no-build, open-and-play design is intentional.
- Stay with vanilla JS and the Canvas 2D API. Don't add frameworks or bundlers without an explicit request.
- Keep the IIFE wrapper and `'use strict'`. Game constants belong in the `DIFFICULTIES` / top-of-script constant blocks rather than being scattered as magic numbers.
- Preserve accessibility hooks: the `#srLive` aria-live region is used by `announce()`, and reduced-motion must continue to gate screen shake, flashes, and pulses.
- Preserve mobile support: the layout uses `touch-action: none`, safe-area insets, and a `resize()` handler. Test any HUD or layout change at phone width.
- Keep the `localStorage` keys stable; changing them resets players' saved best score and settings.

## Verification

There are no automated tests. For gameplay or UI changes, load `index.html` in a browser and check:

1. Start, pause/resume, and game-over flows still work (including the "New Best" and stats display).
2. Each difficulty button sets the expected starting lives and bombs.
3. Weapon power-ups, bomb use, and boss stage transitions behave as before.
4. The layout holds at desktop and phone-sized viewports.

Playwright is available in the cloud environment (`require('playwright')`, Chromium pre-installed) for scripted checks or screenshots.

## Git

- Development branch for this repo: `claude/gifted-tesla-bhvzl1`. Commit and push there; don't push to other branches without permission.
- Don't open pull requests unless the user asks for one.
