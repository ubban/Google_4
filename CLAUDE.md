# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`. There is no build step, no package manager, and no external assets. Everything lives in:

- `index.html` (~1,670 lines): markup, inline CSS (`<style>`), and one inline `<script>` IIFE containing all game logic.
- `README.md`: a placeholder title only.

Open `index.html` directly in a browser to run the game.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The entire game: HTML overlay/HUD, CSS, and JS |
| `README.md` | Minimal project title |

There are no tests, no lint configuration, and no `package.json`. Do not add tooling unless asked.

## Architecture of `index.html`

The script is wrapped in `(function () { 'use strict'; ... })();` so nothing leaks to `window`. Keep new code inside that IIFE.

Key sections, in file order (search for the `// ----------` banners):

- **DOM refs and constants**: element lookups at the top; `STORAGE_*` localStorage keys (`skyfighter.*`).
- **Adaptive performance** (`trackPerf`): samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to keep the frame rate up.
- **Tuning tables**: `DIFFICULTIES` (easy/normal/hard), `THEMES` (six background palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses), and `KILLS_PER_STAGE` (18).
- **Audio**: a WebAudio synth (`ensureAudio`, `beep`, `noiseBurst`). Sounds are generated at runtime; there are no audio files. `AudioContext` is created lazily on first user gesture.
- **Input**: `keys` map keyed by `e.code`; touch joystick and fire/bomb buttons; mouse/touch pointer mapping through `canvasPointFromEvent`.
- **Game state and loop**: `resetGame`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`. `dt` is clamped to 50 ms. Updates run only when `running && !paused`.
- **Combat and spawning**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `addScore` and the combo chain (`breakChain`).
- **Boss patterns**: `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) selected by `updateBoss`.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`. All drawing is 2D canvas primitives; the canvas uses a fixed logical size `BASE_W` x `BASE_H` (480 x 720) scaled by `resize()`.
- **Persistence**: best score in `localStorage`; mute, difficulty, and reduced-motion preferences are also persisted. Every `localStorage` access should tolerate being unavailable (wrap in `try/catch` if you add new ones).

### Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrows or WASD | Left-side joystick |
| Fire (hold) | Space | Fire button |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |

## Conventions

- Match the existing style in `index.html`: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and short section banners in the form `// ---------- Name ----------`.
- Keep the game self-contained: no external scripts, fonts, or images. Visuals are canvas primitives and audio is synthesized.
- Preserve accessibility hooks: announce state changes through `announce()` (the `#srLive` live region), and honor `reducedMotion` for screen shake and flashes (`screenShake`, `flashHit`, `flashBomb`).
- Preserve mobile support. Keep `touchControls` working and avoid hover-only interactions.
- When adding a gameplay value, put it in the relevant tuning table (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, etc.) rather than hardcoding it in logic.
- Use `Math.hypot`, `dt`-scaled movement, and the existing helpers (`rectsOverlap`, `spawnExplosion`) rather than duplicating them.

## Working on this repo

- **Verify changes in a browser.** There are no automated tests. Open `index.html` (or serve it with any static server) and play through the affected path: start, fire, take a hit, use a bomb, boss fight, pause, and game over. Check the browser console for errors.
- **Check both input modes** when touching input code: keyboard and touch.
- **Check reduced motion** when touching effects.
- Keep changes focused. Do not reformat unrelated parts of `index.html`, since the whole game is one file and noisy diffs are hard to review.

## Git

- Development branch for this work: `claude/gifted-tesla-82getf`. Commit and push there.
- Remote: `origin` → `https://github.com/ubban/Google_4`.
- Do not open pull requests unless explicitly asked.
