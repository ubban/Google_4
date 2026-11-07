# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a single-page HTML5 canvas game.

- Everything lives in `index.html`: markup (`<body>`), CSS (`<style>`, roughly lines 10–360), and one inline `<script>` IIFE (starting around line 422) with all game logic, rendering, audio, and input.
- `README.md` is a placeholder and contains no setup instructions.
- There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install is needed.

## Code structure (inside the `<script>` IIFE)

- **Config tables**: `DIFFICULTIES` (easy/normal/hard tuning: spawn rate, enemy speed, enemy fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, and `THEMES` (per-stage sky palettes and names, cycled by `(stage - 1) % THEMES.length`).
- **Persistence**: `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Reads and writes are direct, so keep the key names stable or players lose saved data.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`) with synthesized sound effects (`beep`, `noiseBurst`, and the `sfx` object). Nothing is loaded from files.
- **Game loop**: `loop(now)` → `update(dt)` → `draw()`. `dt` is clamped to 0.05 s. `trackPerf(dt)` feeds performance tracking.
- **Gameplay systems**: `fireWeapon` (Vulcan/Laser, 3 power levels), `useBomb`, `spawnEnemy`, `spawnBoss` and the boss pattern functions (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, and others), `applyPowerUp`, the kill-chain multiplier (`addScore`, `breakChain`), `loseLife`, `endGame`, and `togglePause`.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. HUD elements are DOM nodes (`#hud`, `#bossBarWrap`, `#comboPop`, `#stageBanner`) updated from script.
- **Accessibility**: `announce()` writes to the `#srLive` aria-live region. Reduced motion (`body.reduced-motion`) disables screen shake, flashes, and pulses. It defaults to the `prefers-reduced-motion` media query and can be toggled in the UI.
- **Input**: keyboard (arrows/WASD, Space fire, X/Shift bomb, P/Esc pause), mouse (steer and click to fire), and touch (joystick zone, FIRE and BOMB buttons). Touch handling is in the `joystick*` and `canvasPointFromEvent` helpers.

## Conventions

- Keep the game as one self-contained file unless the user asks for a split. Do not introduce external dependencies or CDN scripts.
- Preserve the existing style: 2-space indentation, single quotes in JS, `const`/`let`, and small named functions inside the IIFE. Do not wrap the game in modules or classes unless asked.
- Tune gameplay through the `DIFFICULTIES` and `THEMES` tables and the constants near them, not by scattering magic numbers.
- Any new user-facing state that should survive a reload needs a `skyfighter.*` localStorage key, and it should be read with a fallback like the existing ones.
- Any new HUD or overlay text should also go through `announce()` where it matters to screen-reader users.
- Keep the layout working on phones (`viewport-fit=cover`, safe-area insets, `touch-action: none` on the canvas) and check touch controls when changing input code.

## Verifying changes

There are no automated tests. For changes to gameplay or UI:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage, a boss fight, a life loss, and a game over on the affected difficulty.
3. For UI changes, check both desktop and a narrow (phone-width) viewport.

## Git

- Develop on the branch the session specifies. Do not push to other branches without explicit permission.
- Write clear commit messages that say what changed and why.
