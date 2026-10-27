# CLAUDE.md

Guidance for working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game lives in one self-contained file, `index.html` (about 1,670 lines: inline CSS, markup, and a single inline `<script>` IIFE). `README.md` is a placeholder.

There is no build step, package manager, bundler, test suite, or linter. To run the game, open `index.html` in a browser or serve the directory with any static file server.

## Repository layout

- `index.html`: the complete game (markup, styles, and logic).
- `README.md`: minimal placeholder.
- `CLAUDE.md`: this file.

## Code structure of `index.html`

- **`<head>`**: meta tags, theme color, and an inline SVG favicon as a `data:` URI.
- **`<style>`** (around lines 10–360): all styling. `color-scheme` is `dark`. The HUD, overlay, boss bar, and touch controls are positioned absolutely inside `#container`.
- **Markup** (around lines 362–420): `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`), boss bar, combo and stage banners, top-bar buttons (mute, reduced motion, pause), touch controls, and the start/game-over `#overlay`.
- **Script** (starts around line 422, wrapped in an IIFE): everything runs inside one closure. Sections, in file order:
  1. DOM references and `STORAGE_*` keys.
  2. Adaptive quality (`quality`, `trackPerf`). Rolling frame-time average lowers or raises a quality level (0 full, 1 medium, 2 low) that controls shadow blur, particle counts, and glow.
  3. Config tables: `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  4. Persisted settings: best score, mute, difficulty, and reduced motion, all via `localStorage`.
  5. Audio: lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and the `sfx` table. Audio stays silent when `muted` is set.
  6. HUD and feedback helpers: hearts, bombs, combo pop, stage banner, screen shake, and screen flash.
  7. Input: keyboard (`keys`), virtual joystick and fire/bomb buttons for touch, and mouse steering and firing on the canvas.
  8. Game state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus scalars such as `score`, `lives`, `stage`, `chain`, and `multiplier`. `resetGame()` reinitializes them.
  9. Gameplay: `spawnEnemy`, `spawnBoss`, `fireWeapon` (vulcan, plus other weapon types), `update(dt)`, `applyPowerUp`, the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireCross`, and others), `updateBoss`, `killBoss`, `loseLife`, `useBomb`, `endGame`, and `togglePause`.
  10. Rendering: `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
  11. Main loop: `loop(now)` clamps `dt` to 50 ms, updates and draws only while `running && !paused`, then calls `requestAnimationFrame`.

## Game design notes

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal`, and `hard` control spawn rate, enemy speed, enemy shot chance, starting lives and bombs, and boss HP multiplier. The selection persists in `localStorage`.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills and then spawns a boss. Themes and bosses cycle with the stage number. Boss HP is `(70 + stage × 35)` scaled by the boss and difficulty multipliers.
- **Enemies**: `grunt` (from stage 1), `interceptor` (from stage 1, weaves), and `tank` (from stage 2, 4 HP).
- **Power-ups**: `life`, `bomb`, and `weapon`, dropped probabilistically by `maybeSpawnPowerUp`. Weapon level caps at 3.
- **Scoring**: kills award points scaled by a chain multiplier. Taking damage breaks the chain.
- **Persistence**: best score is saved to `localStorage` under `skyfighter.bestScore` when a run ends with a new high score.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a split. Do not add external dependencies or CDN scripts without a reason.
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const` by default, and terse inline comments only where the logic is not obvious.
- Add new tunable values to the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) instead of hard-coding them in logic.
- Accessibility features are part of the game. Preserve them when changing the UI:
  - Announce state changes through the `announce()` helper, which writes to the `#srLive` live region.
  - Respect `reducedMotion` (screen shake, flashes, and pulses) and its toggle button.
  - Keep the `aria-label`/`title` attributes on icon buttons.
- Keep touch and keyboard controls working together. Any new action needs a keyboard binding and, where relevant, a touch control.
- Persistent settings go through the `STORAGE_*` constants. Wrap new storage keys in the same pattern rather than inventing new ones.

## Verification

No automated tests exist. After changing gameplay or UI, verify by opening `index.html` in a browser and checking:

- Start, pause (`P`/`Esc`), game over, and restart.
- Keyboard controls (arrows/WASD, Space, X or Shift for bomb) and mouse steering and firing.
- Touch layout at phone width, using a browser device emulator.
- Mute, reduced-motion, and difficulty toggles persist across reloads.
- Browser console shows no errors.

## Git

- Develop on the branch assigned for the task, and push with `git push -u origin <branch>`.
- Commit messages should be short, imperative, and describe the change.
