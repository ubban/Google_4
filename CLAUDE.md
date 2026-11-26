# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo holds **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file, `index.html`, with inline CSS and an inline `<script>` that draws everything on a `<canvas>`. There is no build step, no package manager, and no external dependencies.

Other files:

- `README.md` is a placeholder (title plus a stray timestamp). It has no real project documentation.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There is no install, build, lint, or test command. Verify changes by playing the game in a browser.

## Code layout (`index.html`)

- **Head / CSS** (roughly lines 1–360): meta tags, an inline SVG favicon, and styles for the HUD, overlays, vignette, and touch controls. Colors and sizing use `clamp()` and `vw`, so the layout works at phone width.
- **Markup** (roughly lines 360–420): `#gameCanvas`, the overlays, the HUD elements (`#score`, `#hearts`, `#bombs`, `#weaponName`, and so on), and the touch controls (`#moveZone`, `#fireBtn`, `#bombBtn`).
- **Script** (starts around line 422): one IIFE with `'use strict'`. Key sections, in order:
  - Element references and `STORAGE_*` keys.
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage, cycled), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses, each with a `hpMul`).
  - Audio: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). Sounds are generated, not loaded from files.
  - Rendering helpers: `renderHearts`, `renderBombs`, `showStageBanner`, `screenShake`, `drawPlane`, `drawBossShip`, `draw`.
  - Input: keyboard handlers, a virtual joystick for touch, and mouse or pointer handling on the canvas.
  - Gameplay: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `update(dt)`, `applyPowerUp`, `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
  - Main loop: `loop(now)` is a `requestAnimationFrame` loop. `dt` is capped at 0.05 s. `trackPerf` adjusts a `quality` level (0–2) based on frame time.

## Game rules and controls

- **Keyboard:** arrow keys or WASD move, Space fires, X or Shift drops a smart bomb, P or Escape pauses.
- **Touch:** joystick on the left, fire and bomb buttons on the right.
- **Mouse:** the plane follows the cursor on the canvas, and clicking fires.
- **Weapons:** Vulcan and Laser, each with levels 1–3. A weapon power-up raises the level. Past level 3 it switches weapon type and resets the level to 1.
- **Power-ups:** `life`, `bomb`, and `weapon`.
- **Difficulty:** easy, normal, and hard. These change spawn rates, enemy speed, shooting chance, starting lives and bombs, and boss HP.
- **Stages:** every `KILLS_PER_STAGE` (18) kills triggers a boss fight. Bosses and themes cycle by `(stage - 1) % length`, so they repeat after six stages.
- **Score:** a combo multiplier, which resets on `breakChain`.

## Persistence

The game stores settings in `localStorage` under keys prefixed with `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reduced motion defaults to the `prefers-reduced-motion` media query until the player sets it. Keep new persistent settings under the same `skyfighter.` prefix.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not add a bundler, framework, or npm dependency without asking.
- Stay in the existing style: vanilla JS, 2-space indent, `const` and `let`, and named functions inside the IIFE.
- Use `dt`-scaled movement (pixels per second times `dt`) so gameplay speed does not depend on frame rate.
- Respect the `reduced-motion` class and the `quality` level when adding visual effects. Add effects that are cheap at low quality, or skip them there.
- Use the `announce()` helper to update the `aria-live` region for significant events, such as stage changes or game over.
- Wrap all `localStorage` access in a way that still works when storage is unavailable, since private windows can throw.
- Design for phone-width viewports. Keep touch targets large and don't rely on hover.

## Working in this repo

- Branch: development happens on `claude/gifted-tesla-w6ujk4`. Do not push to other branches without explicit permission.
- Before committing, load `index.html` in a browser (or run a quick syntax check on the inline script) and play through at least one stage and one boss fight if the change touches gameplay.
- Commit messages should describe the gameplay or UI change, not the implementation detail alone.
