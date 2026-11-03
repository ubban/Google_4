# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas game.

| File         | Purpose                                                                 |
|--------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: CSS (inline `<style>`, lines 10–359), markup (lines 360–421), and JavaScript (inline `<script>`, lines 422–1667). |
| `README.md`  | Placeholder title only.                                                  |

There is no build step, no package manager, no bundler, no test suite, and no linter
configuration. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server
(for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000`).

## Architecture of `index.html`

The file is organized top to bottom as:

1. **`<head>`**: viewport meta tuned for mobile (no user scaling, `viewport-fit=cover`),
   theme color, and an inline SVG favicon.
2. **`<style>`**: all styling. HUD is a pill-style bar, the game-over/start UI is a
   glassmorphic overlay card, and there are touch-control styles plus a
   `body.reduced-motion` class that turns off shake, flashes, and pulses.
3. **Markup**: `#container` holds the `<canvas id="gameCanvas">`, the HUD (score, best,
   multiplier, weapon, hearts, bombs), the boss health bar, combo and stage banners,
   the top-right buttons (mute, reduced motion, pause), the touch controls (joystick,
   FIRE, BOMB), the start/overlay card, and an `aria-live` region (`#srLive`) for
   screen readers.
4. **`<script>`**: one IIFE (`'use strict'`) containing the whole game. Sections,
   in order:
   - DOM lookups and `STORAGE_*` keys (`skyfighter.*` in `localStorage`).
   - Adaptive quality: `trackPerf` samples frame times and steps `quality` (0–2)
     down when frames are slow. Shadow blur, explosion particle count, and glow
     check `quality`, so lower quality trades visuals for speed.
   - Tuning tables: `DIFFICULTIES` (easy / normal / hard), `KILLS_PER_STAGE`,
     `THEMES` (sky palettes), `ENEMY_TYPES`, `BOSSES`.
   - Audio: a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`
     and a named `sfx` object. Audio is created on the first user gesture.
   - HUD helpers: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`,
     `showComboPop`, `showStageBanner`, `screenShake`.
   - Input: keyboard (`keys[e.code]`), a touch joystick (`joystickStart/Move/End`),
     and mouse steering on the canvas.
   - Game state: `resetGame`, the entity spawners (`spawnEnemy`, `spawnBoss`,
     `maybeSpawnPowerUp`, `spawnExplosion`), and actions (`fireWeapon`, `useBomb`,
     `addScore`/`breakChain` for the kill-chain multiplier).
   - Simulation: `update(dt)` advances everything; `applyPowerUp` handles `life`,
     `bomb`, and `weapon` pickups.
   - Bosses: `BOSS_PATTERNS` is a list of bullet-pattern functions (radial, aimed
     spread, columns, spiral, cross, seeded homing, double ring). `updateBoss` and
     `killBoss` drive them.
   - Flow: `loseLife`, `endGame` (saves best score), `togglePause`.
   - Rendering: `drawPlane`, `drawBossShip`, `drawClouds` (parallax layers), `draw()`.
   - Main loop: `loop(now)` clamps `dt` to 50 ms, then calls `update` and `draw` while
     running and not paused. Event wiring for buttons and input is at the end of the IIFE.

### Game rules to keep in mind

- Starting lives and bombs come from the selected difficulty. Hearts and bomb counts
  are capped (`maxLives`, `maxBombs`).
- A stage ends after `KILLS_PER_STAGE` kills, then a boss fight begins. The boss design
  cycles through `BOSSES` by stage, and its HP is `(70 + stage * 35)` times the boss's
  `hpMul` and the difficulty's `bossHpMul`. Enemy types gate on `minStage`.
- Power-up drop odds (in `maybeSpawnPowerUp`): 6% extra life, 8% bomb, 20% weapon.
  Weapons are Vulcan or Laser, each with levels 1–3. Picking up a weapon past level 3
  swaps the weapon type and resets the level to 1.
- Audio starts only after a user gesture. Keyboard input, the touch FIRE button, and the
  start button call `ensureAudio()`, which must run before any sound plays.

### Persistence

Only four `localStorage` keys are used: `skyfighter.bestScore`, `skyfighter.muted`,
`skyfighter.difficulty`, and `skyfighter.reducedMotion`. Reads and writes are not wrapped
in `try/catch`, so if you add more storage, guard it the same way the existing code would
need to be guarded (storage can throw in private windows or when site data is blocked).

## Conventions

- **Single file.** Keep the game in `index.html`. Do not split out CSS or JS files
  unless the user asks, since the project is deliberately dependency-free.
- **No dependencies.** Do not add libraries or CDN scripts without an explicit request.
- **Keep the IIFE.** New code belongs inside the existing `(function () { ... })()` so it
  does not leak globals. Use `const`/`let` and `'use strict'`-compatible code.
- **Tune in tables.** New enemies, bosses, themes, and difficulty values should be added
  to the existing tables (`ENEMY_TYPES`, `BOSSES`, `THEMES`, `DIFFICULTIES`) rather than
  hard-coded into logic.
- **Respect reduced motion.** Any new screen shake, flash, or pulse effect should go
  through the existing `reducedMotion` check and the `screenShake` / `flash*` helpers.
- **Keep it accessible.** Announce important state changes through `announce()` so they
  reach the `aria-live` region. Keep controls labeled (`title` / text content).
- **Mobile first.** Verify changes at phone width. The layout uses `resize()` and
  `BASE_W` / `BASE_H` (480×720) as the logical play area, scaled to the viewport.

## Verifying changes

There are no automated tests. For changes to gameplay or UI:

1. Open `index.html` in a browser (or serve it statically) and play a run.
2. Check the affected flow end to end: start, the change itself, pause/resume, game over,
   and restart. Confirm the best score persists across reloads.
3. For layout work, check both desktop and a narrow phone-sized viewport, including the
   touch controls.
4. Watch the browser console for errors. A JavaScript error stops the loop silently
   because `loop` has no error handling.

Also run a syntax check on the inline script if you have Node available. For example,
extract the `<script>` block to a temporary file and run `node --check` on it.

## Git workflow

- Develop on the branch the task specifies (currently `claude/gifted-tesla-vazstg`).
- Commit with clear messages that describe the change, and push with
  `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
