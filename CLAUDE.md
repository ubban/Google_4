# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- Entire game lives in one file: `index.html` (inline `<style>` and one inline `<script>` IIFE, ~1,670 lines).
- No build step, no package manager, no dependencies, no test suite. Open the file in a browser to play.
- `README.md` is a placeholder and contains no setup instructions.

## Running locally

```bash
# any static server works; or just open the file directly
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code layout (`index.html`)

- **Markup**: `#container` holds the `<canvas id="gameCanvas">`, HUD (`#hud`), boss bar, combo/stage popups, top-bar buttons (mute, reduced-motion, pause), touch controls, and the `#overlay` start/game-over card.
- **Script**: a single `(function () { ... })()` IIFE. Sections, roughly in order:
  1. DOM references (`const xxxEl = document.getElementById(...)`).
  2. Persistence keys (`skyfighter.*` in `localStorage`) and settings.
  3. Config tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
  4. Audio: lazily-created `AudioContext` via `ensureAudio()`, with `beep()`, `noiseBurst()`, and the `sfx` map.
  5. HUD and effects helpers (`renderHearts`, `renderBombs`, `flashHit`, `screenShake`, `showStageBanner`, ...).
  6. Scaling: `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the window (max 1.6x).
  7. Input: keyboard (`keys`), mouse (`mouseTarget`), virtual joystick and buttons for touch.
  8. Game state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus score/lives/bombs/stage/chain/multiplier.
  9. `resetGame()`, `update(dt)`, `updateBoss(dt)`, `draw()`, `drawPlane()`, `drawBossShip()`, `loop(now)`.
- **Adaptive quality**: `trackPerf(dt)` lowers `quality` (0 full, 1 medium, 2 low) when frame time averages poorly; draw code reads `quality` to reduce shadows, glow, and particle counts.

## Game rules (current behavior)

- **Controls**: mouse move/click to steer and fire; arrows/WASD move; Space fires; X or Shift drops a bomb; P or Esc pauses. Touch: drag left to fly, FIRE and BOMB buttons.
- **Difficulty** (`DIFFICULTIES`): Easy / Normal / Hard change spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Stages**: each stage requires `KILLS_PER_STAGE` (18) kills, then a boss fight. `THEMES` cycles the sky palette per stage.
- **Enemies**: `grunt` (1 HP), `interceptor` (fast, weaving), `tank` (4 HP, from stage 2).
- **Bosses**: `BOSSES` array with per-boss colors and `hpMul`; they telegraph attacks and show a health bar.
- **Weapons**: Vulcan (default) and Laser; a power-up toggles `player.weapon` between them. `player.level` tracks weapon power and is shown in the HUD. Laser fires every 0.1 s, Vulcan every 0.15 s.
- **Bombs**: limited count, clear the screen and damage bosses.
- **Scoring**: kill-chain multiplier (`chain`, `multiplier`). Each kill refreshes `chainTimer` (2.2 s); the chain breaks when it runs out.
- **Persistence**: best score in `localStorage` under `skyfighter.bestScore`; mute, difficulty, and reduced-motion preferences under `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Reads are wrapped so the game still works if storage is unavailable.
- **Accessibility**: `#srLive` announces key events via `announce()`; a reduced-motion toggle disables screen shake, flashes, and pulses (CSS class `body.reduced-motion`).

## Conventions

- Keep the game as a **single self-contained file** unless the user asks for a restructure. Do not introduce a bundler, framework, or external runtime dependencies.
- Keep all new tunables in the existing config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding magic numbers in update/draw code.
- Keep the game loop allocation-light: reuse arrays and avoid creating objects per frame in hot paths.
- Preserve mobile behavior: the layout must work at phone width, touch controls only show on touch devices, and `touch-action: none` stays on the canvas and body.
- Respect `reducedMotion` and `quality` when adding visual effects.
- Use `announce()` for any new state change that a screen-reader user should hear.
- Match the existing style: 2-space indent, single quotes in JS, `const`/`let`, semicolons at statement ends.

## Verifying changes

There are no automated tests. Before reporting a change as done:

1. Load `index.html` in a browser (desktop and a narrow/mobile viewport) and check the browser console for errors.
2. Play through at least one full stage to a boss, and check game over and restart.
3. For HUD or layout changes, check at phone width (no horizontal scroll, HUD wraps cleanly).
4. For audio changes, confirm nothing throws when muted and when the audio context is suspended.

If you cannot run the game in a browser, say so explicitly and limit claims to what you verified by reading the code.

## Git workflow

- Develop on the branch the task specifies (currently `claude/gifted-tesla-3fe4nx`); do not push to `main` without explicit permission.
- Commit with clear, descriptive messages that explain the gameplay or UI change.
- Do not open a pull request unless the user asks for one.
