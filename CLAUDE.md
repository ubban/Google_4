# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project. The only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up, shipped as a single self-contained file:

- `index.html` (~1,670 lines): all markup, CSS, and JavaScript, with no build step, no dependencies, and no package manager.
- `README.md`: a placeholder title only.

There is no test suite, linter, bundler, or CI configuration. Verify changes by opening `index.html` in a browser (or serving the directory with any static server) and playing.

## File layout of `index.html`

| Lines (approx.) | Section |
| --- | --- |
| 1–9 | `<head>`: viewport (no zoom), theme color, inline SVG favicon |
| 10–360 | `<style>`: dark-only UI (`color-scheme: dark`), HUD, overlay, touch controls, responsive scaling |
| 361–421 | `<body>`: `#container` with canvas, vignette/flash overlays, HUD, boss bar, combo/stage banners, touch controls, start/pause overlay with difficulty buttons |
| 422–end | `<script>`: one IIFE containing the entire game |

Inside the IIFE, the code is grouped roughly as follows. Search for these names when navigating:

- **DOM refs and constants**: `STORAGE_*` keys, `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (6 designs, cycled by stage), `KILLS_PER_STAGE`.
- **Persistence and accessibility**: best score, mute, difficulty, and reduced-motion preference are stored in `localStorage`. Reduced motion defaults to the `prefers-reduced-motion` media query. `announce()` writes to the `#srLive` region.
- **Quality scaling**: `trackPerf()` watches frame times and drops `quality` (0 full, 1 medium, 2 low), which reduces particle counts and glow.
- **Audio**: lazily created `AudioContext` (`ensureAudio`), with oscillator and noise helpers (`beep`, `noiseBurst`) and an `sfx` table. Nothing plays until a user gesture.
- **Input**: keyboard (`keys` map by `e.code`), touch joystick plus fire/bomb buttons, and mouse (`mouseTarget`, drag-to-move and hold-to-fire). Canvas is letterboxed and scaled in `resize()` against a 480×720 base size.
- **Simulation**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `endGame`, `togglePause`.
- **Boss AI**: `BOSS_PATTERNS[designIndex]` is an array of attack functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss` cycles through them and telegraphs before firing.
- **Rendering**: `draw()` (background, clouds, sprites, particles, HUD bar), `drawPlane`, `drawBossShip`, `drawClouds`. All rendering is Canvas 2D.
- **Loop**: `loop(now)` clamps `dt` to 50 ms, skips update and draw while paused, and runs via `requestAnimationFrame`. The start button is wired near the end of the file.

## Gameplay facts (for keeping docs and tuning consistent)

- Controls: **Arrow keys or WASD** move; **Space** fires; **X** or **Shift** drops a bomb; **P** or **Esc** pauses. Mouse: the ship follows the cursor over the canvas and holding the button fires. Touch: a virtual joystick on the left, with fire and bomb buttons on the right.
- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, shoot chance, starting lives, starting bombs, and boss HP multiplier.
- Power-ups: `life` (6%), `bomb` (8%), `weapon` (20%). Weapon power-ups raise the level (1→3), then toggle between **vulcan** and **laser** and reset to level 1.
- Each stage ends after `KILLS_PER_STAGE` (18) kills and triggers a boss. The theme and boss design change with the stage.
- The chain/combo multiplier breaks when its timer runs out or a life is lost (`breakChain`).

## Conventions

- Keep the game in a single file unless the user asks for a split. Do not add a build tool, framework, or external CDN dependency without being asked.
- Keep the code style already in use: 2-space indentation, single quotes in JS, `const` by default, and tuning values in the `DIFFICULTIES`, `ENEMY_TYPES`, and `BOSSES` tables rather than hard-coded in logic.
- Preserve the existing `localStorage` keys (`skyfighter.*`). Renaming them silently resets players' saved best score and settings.
- Preserve accessibility hooks: the `aria-live` region, `prefers-reduced-motion` handling, and the `sr-only` text.
- Touch and desktop must both keep working. Test changes to input code on both paths when possible.
- Audio must start only after a user gesture (browser autoplay policy). Keep calls to `ensureAudio()` on input handlers.

## Working in this repo

- Branch: develop on the designated feature branch. Do not push to other branches without explicit permission.
- Commit with clear messages. Do not open a pull request unless the user asks for one.
- For manual verification, load `index.html` directly in a browser. The Playwright browser at `/opt/pw-browsers/chromium` is available if automated checks are needed.
