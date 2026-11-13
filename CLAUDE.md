# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`, with inline CSS and one inline `<script>` IIFE. There is no build step, no bundler, no package manager, and no external dependencies.

Repository contents:

- `index.html` — the game (markup, styles, and game logic, ~1,600 lines).
- `README.md` — minimal placeholder.
- `CLAUDE.md` — this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static file server, e.g. `python3 -m http.server` and visit `http://localhost:8000`. No install step is needed.

## Code layout inside `index.html`

- **Head / CSS (lines ~1–360):** dark color scheme, full-viewport canvas (`#gameCanvas`), HUD (`#hud`, `#bossBarWrap`, `#topBar`, `#stageBanner`, `#comboPop`), touch controls (`#touchControls`, `#moveZone`, `#actionZone`), and overlay/menu cards (`#overlay`, `#overlayCard`, `#statsRow`, `#diffRow`).
- **Script (starts at line ~422):** a single `(function () { 'use strict'; ... })()` wrapper. Keep new code inside it; do not add globals.
  - Storage keys (`skyfighter.*`): best score, mute, difficulty, reduced motion.
  - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
  - `THEMES`: background palettes cycled per stage.
  - `ENEMY_TYPES` (grunt, interceptor, tank) and `BOSSES` (six bosses, cycled by stage).
  - Audio: `ensureAudio`, `beep`, `noiseBurst`. Audio is created lazily on the first user gesture.
  - Rendering: `drawPlane`, `drawBossShip`, `draw`, `drawClouds`.
  - Simulation: `update` (per-frame), `fireWeapon` and the pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`), `updateBoss`, `killBoss`, `loseLife`, `useBomb`, `applyPowerUp`, `endGame`, `togglePause`.
  - Main loop: `loop`, driven by `requestAnimationFrame`.

## Controls

- Keyboard: arrow keys or WASD to move; Space to fire; X, Shift to drop a bomb; P or Escape to pause.
- Touch: left-side joystick to move; fire and bomb buttons on the right.
- Motion controls (device motion) are supported where the browser allows them.
- Weapons toggle between VULCAN and LASER via a power-up.

## Conventions

- Keep the game dependency-free. Do not introduce npm packages, CDN scripts, or a build step unless the user asks for one.
- Keep everything in `index.html` unless the user asks for a split. If the file is split later, update this section.
- Preserve the `'use strict'` IIFE structure and the existing naming style (camelCase functions, UPPER_CASE config tables, `STORAGE_*` constants for localStorage keys).
- Persisted settings must go through the existing `STORAGE_*` keys and be wrapped so the game still works when `localStorage` is unavailable.
- Any new gameplay values (spawn rates, HP, scores) belong in the relevant config table (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) rather than as magic numbers in the update loop.
- Respect the reduced-motion setting when adding screen shake, flashes, or other motion effects.
- The layout must keep working on touch devices and in both landscape and portrait viewports; test `touchControls` behavior when changing input handling.

## Testing and verification

There is no automated test suite. Verify changes by:

1. Opening `index.html` in a browser (desktop and, where possible, a mobile-sized viewport).
2. Playing through at least one stage, including a boss fight, a bomb, a weapon power-up switch, and a game-over/restart.
3. Checking the console for errors.
4. Toggling mute, difficulty, and reduced motion, and reloading to confirm they persist.

If you add a test harness later, document how to run it here.

## Git

- Development branch for automated work: `claude/gifted-tesla-hxnonl`.
- Make focused commits with clear messages describing the gameplay or code change.
- Do not open pull requests unless the user asks for one.
