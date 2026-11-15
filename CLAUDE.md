# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

Files:

- `index.html` — the entire game: markup, CSS (in a `<style>` block), and JavaScript (in a single IIFE `<script>` block). ~1,700 lines.
- `README.md` — placeholder title only.

There is no `package.json`, bundler, test suite, linter, or build step. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- To verify a change, load the page and play it. Syntax-check the inline script if you want a quick static check (e.g. extract the `<script>` body and run `node --check`).
- Runtime behaviour (collisions, pacing, boss patterns, touch controls) cannot be verified by reading alone. Say so explicitly if you could not run it.

## Architecture of `index.html`

Everything sits inside one `(function () { 'use strict'; ... })()` closure. Key sections, roughly in file order:

1. **DOM references** — `const xEl = document.getElementById(...)` at the top of the script. Add new HUD/overlay elements here and in the markup.
2. **Persistence keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Changing a key orphans players' saved data; avoid unless intended.
3. **Adaptive quality** — `trackPerf()` steps a `quality` level (0–2) based on a rolling frame-time average.
4. **Tuning tables** — the main place to change game feel:
   - `DIFFICULTIES` — spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier (`easy` / `normal` / `hard`).
   - `KILLS_PER_STAGE` — kills needed to advance a stage.
   - `THEMES` — per-stage sky gradient and cloud/sun colours (cycled by stage).
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP). Fields: size, HP, speed multiplier, colours, score, `minStage`.
   - `BOSSES` — six bosses (VANGUARD → TITAN) with colours and `hpMul`.
5. **Audio** — `ensureAudio()` lazily creates the `AudioContext` (must follow a user gesture). `beep()` and `noiseBurst()` synthesise all sound effects; there are no audio files. `EXPLODE_SFX` maps enemy types to sounds.
6. **HUD and feedback helpers** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, and `announce()` (polite `aria-live` text for screen readers).
7. **Layout and input** — `resize()` scales the canvas to a fixed logical size (`BASE_W` × `BASE_H` = 480 × 720). Touch uses a virtual joystick (`joystick*`) plus FIRE and BOMB buttons. Keyboard, mouse, and pointer events are handled further down.
8. **Game state** — `resetGame()` initialises state; `update(dt)` advances the simulation. `dt` is clamped to 50 ms so tab switches don't teleport objects.
9. **Weapons, bombs, power-ups** — `fireWeapon()` (Vulcan/Laser, 3 power levels), `useBomb()` (screen clear, damages bosses), `applyPowerUp()`, and `maybeSpawnPowerUp()`.
10. **Boss AI** — `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, etc.). `updateBoss(dt)` cycles through them; `killBoss()` advances the stage.
11. **Lifecycle** — `loseLife()`, `endGame()`, `togglePause()`.
12. **Rendering** — `draw()` paints the parallax clouds, sun, planes (`drawPlane`), boss (`drawBossShip`), bullets, and effects. Drawing is pure canvas 2D with no sprite assets.
13. **Main loop** — `loop(now)` runs under `requestAnimationFrame`, calling `update` then `draw` only while `running && !paused`.

Conventions to follow:

- Keep the single-file, no-dependency design. Don't add a framework, a bundler, or external assets without asking.
- Put tunable numbers in the existing config tables rather than inline magic numbers.
- Keep the IIFE closure. Don't leak globals.
- New player-facing state changes should call `announce()` for screen-reader users, and respect the `reducedMotion` setting for visual shake or flash effects.
- Preserve the mobile layout: `touch-action: none`, safe-area insets, and the 16:9-ish fixed logical canvas.

## Controls (for testing)

- **Mouse:** move to steer, click to fire.
- **Keyboard:** Arrows/WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses.
- **Touch:** drag the left zone to fly, tap FIRE, tap BOMB.

## Git workflow

- Development branch: `claude/gifted-tesla-3whyaw`. Commit and push there. Don't push to other branches without explicit permission.
- Don't create pull requests unless explicitly asked.
- Commit messages: imperative summary line, then a short body describing the gameplay or code change.
