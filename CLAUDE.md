# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project containing **Sky Fighter**, a Raiden-style
vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript
  (inline `<script>`, wrapped in an IIFE with `'use strict'`). ~1,700 lines.
- `README.md` — currently a stub (title only).

There is no build step, no package manager, no bundler, and no test suite.
Do not add one unless the user asks for it.

## Running the game

Open `index.html` directly in a browser. No server is required.

## Code structure (inside `index.html`)

The script is organized roughly in this order:

1. **DOM references and constants** — element lookups, `STORAGE_*` keys,
   `DIFFICULTIES` table, `KILLS_PER_STAGE`.
2. **Persistence** — `localStorage` for best score, mute, difficulty, and
   reduced-motion. Every access is a plain call; keep it that way unless a
   change needs guarding.
3. **Audio** — synthesized sound via the Web Audio API (`beep`, `noiseBurst`,
   `ensureAudio`). No audio asset files.
4. **UI helpers** — HUD updates (`renderHearts`, `renderBombs`), flash/shake
   effects, combo pop-ups, stage banner.
5. **Input** — keyboard (`keydown` handler), mouse/pointer, and touch
   joystick + on-screen FIRE/BOMB buttons. Motion controls are optional.
6. **Game state** — `resetGame()` initializes state; module-level `let`
   variables hold score, kills, stage, weapon level, bombs, lives, etc.
7. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
   Drawing is canvas 2D path code; there are no image assets.
8. **Gameplay** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`,
   `fireWeapon`, `useBomb`, `update(dt)`, `applyPowerUp`, boss attack
   patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, etc.),
   `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
9. **Main loop** — `loop(now)` drives `update` and `draw` via
   `requestAnimationFrame`, with `trackPerf` monitoring frame time.

### Key controls (for reference when testing)

- Mouse: move to steer, click to fire.
- Arrows / WASD: move. Space: fire. X or Shift: bomb. P or Esc: pause.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

## Conventions

- **Single file.** Keep CSS and JS inline in `index.html` unless the user asks
  to split them. Do not introduce external dependencies or CDN scripts.
- **No frameworks.** Plain DOM and Canvas 2D APIs.
- **Match existing style:** 2-space indentation, single quotes in JS, `const`
  by default with `let` for mutable game state, small named helper functions.
- **Mobile-aware.** Preserve `touch-action`, safe-area insets, and the
  phone-width layout when changing CSS or input handling.
- **Accessibility.** Respect the `reduced-motion` setting for new animations
  (see existing `body.reduced-motion` rules).
- **Frame-rate independence.** Game updates take a `dt` argument; new movement
  or timers should use it rather than per-frame counts.

## Verification

Because there are no automated tests, verify changes by loading `index.html`
in a browser and playing through the affected path (start, shoot, bomb, stage
transition, boss, game over). For visual changes, check both desktop and a
narrow mobile viewport. Also confirm there are no console errors.

## Git

- Development branch for this session: `claude/gifted-tesla-8hpnfk`.
- Commit with clear messages describing the gameplay or UI change.
- Do not open pull requests unless the user explicitly asks.
