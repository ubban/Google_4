# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is one self-contained file, `index.html`, with inline `<style>` and `<script>` blocks. `README.md` is a placeholder and has no setup steps.

There is no build step, package manager, test suite, linter, or bundler. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static file server, for example `python3 -m http.server`. Nothing needs to be installed.

## File layout

- `index.html`, about 1,670 lines:
  - Lines ~10–362: CSS. Covers the HUD, overlays, the boss bar, touch controls, and `body.reduced-motion` overrides.
  - Lines ~363–421: DOM markup. This covers the canvas `#gameCanvas`, the HUD (`#hud`), the boss bar, the start/pause overlay, and the touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`).
  - Lines ~422–1668: one IIFE (`'use strict'`) that holds all game logic, audio, input, and rendering.
- `README.md`: placeholder.

## Architecture of the game script

The script is organized in sections, marked with `// ---------- Name ----------` comments. Keep new code in the matching section.

- **Config tables** (near the top): `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (six stage skies), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six bosses, each with `hpMul`). Tune balance here, not inside update logic.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Read and write them only through the existing constants.
- **Adaptive quality**: `trackPerf` samples frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low). Respect this when adding particle or visual effects.
- **Audio**: `ensureAudio` creates the `AudioContext` lazily. It must run from a user gesture. Effects are built from `beep` and `noiseBurst`, and `sfx` groups the named sounds. Respect `muted`.
- **Input**: keyboard (`keys` map, `window` keydown/keyup), mouse on the canvas, and touch joystick plus buttons. Keep all three working when you touch input handling.
- **Game state**: `resetGame()` builds `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, and `boss`. `update(dt)` advances the simulation. `draw()` renders it. `loop(now)` calls both each frame and caps `dt` at 0.05 s.
- **Weapons**: `fireWeapon()` implements two weapon types, `vulcan` and `laser`, each with power levels 1–3.
- **Power-ups**: `maybeSpawnPowerUp` drops `life`, `bomb`, or `weapon` pickups. `applyPowerUp` applies them.
- **Bosses**: `spawnBoss`, `updateBoss`, and `killBoss`. Attack patterns are the `fire*` helpers collected in `BOSS_PATTERNS`.
- **Flow**: `loseLife`, `endGame`, and `togglePause` control the run. The overlay (`#overlay`) is used for the start, pause, and game-over screens.
- **Accessibility**: `announce()` writes to the `#srLive` live region. `reducedMotion` (also set from `prefers-reduced-motion`) toggles the `reduced-motion` body class, which disables CSS flashes and pulses, and makes `screenShake` cap shake duration at 0.1 s.

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Don't add external scripts, CDN links, or frameworks without asking.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for callbacks, and short section comments.
- Coordinates are in the fixed base canvas space `BASE_W × BASE_H` (480 × 720). `resize()` scales the canvas to fit the viewport, so don't hardcode pixel sizes that bypass it.
- Keep the game playable with both touch and keyboard. Test changes to controls on both paths.

## Verifying changes

No automated tests exist. To verify a change:

1. Open `index.html` in a browser, or run a static server, and check the browser console for errors.
2. Play through start, a few waves, a boss fight, bomb use, weapon pickups, pause/resume, and game over.
3. Check the mobile layout and touch controls at a phone-sized viewport.
4. Confirm `localStorage` persistence for best score, mute, difficulty, and reduced motion after a reload.

If you change balance values, note the before and after numbers in your summary.

## Git

- Develop on the designated feature branch (currently `claude/gifted-tesla-9ey3g7`), not `main`.
- Keep commits focused. Write messages that describe the behavior change, such as "Add ..." or "Fix ...".
