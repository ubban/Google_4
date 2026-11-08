# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single-file HTML5 canvas game.

| File | Purpose |
|------|---------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`), ~1,670 lines |
| `README.md` | Placeholder readme (title only) |

There is no build step, no package manager, no bundler, no framework, and no test suite.
There is no `package.json`. The game runs by opening `index.html` in a browser.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static
  server (for example `python3 -m http.server` from the repo root, then visit
  `http://localhost:8000`).
- **Tests:** none exist. Verify changes by playing the game in a browser, and check both
  desktop (keyboard/mouse) and touch layouts when touching input or HUD code.
- **Lint/build:** none configured. Keep the JavaScript valid for modern browsers
  (ES2017+ syntax is used; no transpilation).

## Architecture of `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Everything is private
to that closure, and nothing is exposed on `window`. Keep new code inside it.

Major sections, in file order:

1. **DOM references**: `const x = document.getElementById(...)` at the top of the script.
   The HTML element IDs are referenced here, so renaming an element ID requires updating
   this block.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`
   (all prefixed `skyfighter.`). Changing a key orphans existing players' saved data.
3. **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and an `sfx` object of named
   sound effects. Audio is created lazily on the first user gesture (`ensureAudio()` is
   called from `keydown`).
4. **Tuning tables**:
   - `DIFFICULTIES`: `easy` / `normal` / `hard`, with spawn rates, enemy speed, shoot
     chance, starting `lives`, starting `bombs`, and `bossHpMul`.
   - `KILLS_PER_STAGE`: kills required before a boss fight.
   - `THEMES`: sky gradient palettes, one per stage (cycled).
   - `ENEMY_TYPES`: `grunt`, `interceptor`, `tank`, with `minStage` gating when each appears.
   - `BOSSES`: six boss designs, each with colors and an `hpMul`.
5. **Canvas sizing**: fixed logical resolution `BASE_W × BASE_H` = `480 × 720`, scaled to
   fit the window (capped at 1.6×). Game coordinates always use the 480×720 space.
6. **Input**: keyboard (`keys` map keyed by `e.code`), mouse/pointer steering and firing
   on the canvas, and touch joystick (`joystickStart/Move/End`) plus FIRE and BOMB buttons.
7. **Game state and lifecycle**: `resetGame()`, `endGame()`, `loseLife()`, `togglePause()`.
   Persisted best score is written only in `endGame()`.
8. **Spawning and combat**: `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`,
   `fireWeapon()`, `useBomb()`, `addScore()` (with `breakChain()` for the kill-chain
   multiplier).
9. **Update loop**: `update(dt)` advances all simulation state. `applyPowerUp(type)`
   handles `life`, `bomb`, and `weapon` pickups.
10. **Boss patterns**: `updateBoss()` and the pattern emitters `fireRadial`,
    `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`,
    `fireDoubleRing`. `killBoss()` advances `stage`.
11. **Rendering**: `draw()` plus helpers `drawPlane`, `drawBossShip`, `drawClouds`.
    `draw()` runs once per frame after `update()`.
12. **Main loop**: `loop(now)` drives `update` and `draw` through `requestAnimationFrame`
    and feeds `trackPerf()`. `trackPerf()` adaptively lowers `quality` (0 = full,
    1 = medium, 2 = low) when frame times are slow. Quality reduces shadow blur, particle
    counts, and glow.

### Game state conventions

- Time-based motion uses `dt` passed into `update(dt)` (seconds). `loop()` clamps `dt` to
  0.05 s, so long stalls (tab switches, GC pauses) do not teleport entities. Do not assume
  a fixed frame rate.
- Entity lists (enemies, bullets, power-ups, particles) are plain arrays mutated in place.
  Use `rectsOverlap(a, b)` for collision checks.
- HUD updates go through helper functions (`renderHearts`, `renderBombs`, `flashHit`,
  `showComboPop`, `showStageBanner`). Prefer these over writing DOM directly from game logic.
- Accessibility: announce important state changes through `announce(text)`, which writes
  to the `#srLive` live region. Respect the reduced-motion setting (`reducedMotion`) for
  screen shake, flashes, and pulses.
- Audio and visual effects should stay gated by `muted` and `quality` where they already are.

## Persistence

All persistence is `localStorage`, read and written with direct calls, not wrapped in
try/catch. Keys:

- `skyfighter.bestScore`: number, written on game over
- `skyfighter.muted`: `'1'` or `'0'`
- `skyfighter.difficulty`: `'easy'`, `'normal'`, or `'hard'`
- `skyfighter.reducedMotion`: `'1'` or `'0'`. If absent, the `prefers-reduced-motion` media
  query decides the default.

If you add storage, follow the `skyfighter.` prefix convention and add the key to this list.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a split.
  Do not add external CDN scripts or fonts without a reason. The page uses system font
  stacks and inline SVG/data-URI assets.
- Preserve the mobile-first behavior: `touch-action: none`, `viewport-fit=cover`,
  safe-area insets in the HUD, and the touch control overlay that appears only on touch
  devices (`isTouchDevice`).
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const`/`let`
  (no `var`), arrow functions for callbacks, and short section banners of the form
  `// ---------- Name ----------`.
- Keep the `controlsHint` text in the overlay in sync with the actual bindings when controls
  change (keyboard: Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause; mouse:
  move to steer, click to fire).

## Git workflow

- Feature work goes on the designated development branch, never directly on the default branch.
- Commit messages in this repo use an imperative summary line followed by a bulleted body
  describing the change (see `git log`).
- Do not open pull requests unless the user explicitly asks for one.
