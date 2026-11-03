# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. Everything lives in `index.html` (~1,670 lines): inline CSS in `<style>`, markup, and one IIFE-wrapped inline `<script>`. There is no build step, no package manager, no bundler, no framework, and no test suite.

Other files:
- `README.md` — a one-line title; it contains no setup or usage instructions.
- `.git/` — the repository. Current history: `Initial commit`, then `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.

## Running and testing

- **Run:** open `index.html` directly in a browser. No server is needed.
- **Build / lint / test:** none exist. Do not invent commands for them. Verify changes by loading the page and playing, or with a headless browser (Playwright is preinstalled in the cloud environment; see the environment notes).
- Keep the file self-contained. Do not add external script or stylesheet dependencies unless the user asks for them.

## Code layout inside `index.html`

1. **`<head>`** — viewport meta (locks zoom, `viewport-fit=cover`), `theme-color`, an inline SVG favicon, and the `<style>` block.
2. **CSS** — dark color scheme (`color-scheme: dark`). Sections cover the full-screen container, the `#gameCanvas` element, the HUD (`#hud`, `.stat` pills), the top bar buttons (`#topBar`, `.iconBtn`), the overlay card (`#overlay`), difficulty buttons (`.diffBtn`), touch controls (`#touchControls`, joystick, FIRE/BOMB), the boss health bar, and screen-flash/vignette elements.
3. **Markup** — the canvas, HUD spans (`score`, `best`, `mult`, `weaponName`, `weaponLvl`, `hearts`, `bombs`), transient popups (`comboPop`, `stageBanner`), accessibility live region (`srLive`), touch controls, and the start/title overlay.
4. **Script** — one `(function () { 'use strict'; ... })();` IIFE. Its sections, in order:
   - DOM lookups (`const xxxEl = document.getElementById(...)`).
   - Constants: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
   - Persisted settings loaded from `localStorage`.
   - Audio: a lazily created `AudioContext` (`ensureAudio`), `beep` / `noiseBurst` primitives, and a named `sfx` map.
   - HUD helpers: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
   - Responsive scaling: `BASE_W = 480`, `BASE_H = 720`, `resize()` scales the canvas up to 1.6×.
   - Input: keyboard (`keys` map keyed by `e.code`), touch joystick and buttons, and mouse steering/firing on the canvas.
   - Game state (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`) and `resetGame()`.
   - Drawing: `drawPlane`, `drawBossShip`, `draw()`, `drawClouds`.
   - Simulation: `update(dt)`, `fireWeapon()`, `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `updateBoss(dt)`, and boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
   - Scoring: `addScore` (chain multiplier), `breakChain`, `useBomb`, `endGame`.
   - Main loop: `loop(now)` calls `update` then `draw` via `requestAnimationFrame`, with `dt` clamped to 50 ms and a perf tracker (`trackPerf`) that adjusts `quality` (0 full, 1 medium, 2 low) to reduce particle counts.

## Gameplay rules (as implemented)

- **Difficulty** (`DIFFICULTIES`): Easy, Normal (default), Hard. Each sets spawn rate, enemy speed, enemy fire chance, starting lives (4 / 3 / 3), starting bombs (3 / 2 / 2), and a boss HP multiplier (0.8 / 1.0 / 1.3).
- **Enemies** (`ENEMY_TYPES`): `grunt` (1 HP, 10 pts, stage 1+), `interceptor` (1 HP, 15 pts, weaves, stage 1+), `tank` (4 HP, 35 pts, stage 2+).
- **Stages:** each stage requires `KILLS_PER_STAGE` (18) kills, then a boss spawns after a short delay. Bosses cycle through `BOSSES` by `(stage - 1) % 6`.
- **Boss HP:** `round((70 + 35 × stage) × difficulty.bossHpMul × boss.hpMul)`.
- **Weapons:** Vulcan (default, 0.15 s fire delay) and Laser (0.10 s). Weapon power-ups toggle between the two; `level` is shown in the HUD.
- **Power-up drops** (`maybeSpawnPowerUp`): on a random roll, 6% extra life, 8% bomb, 20% weapon switch. Bomb (`useBomb`) clears enemy bullets, destroys all on-screen enemies (5 pts each), and deals 24 damage to the boss.
- **Scoring:** points are multiplied by a chain multiplier, `1 + min(9, floor(chain / 5))`. The chain resets after 2.2 s without a kill.
- **Best score** is saved in `localStorage` under `skyfighter.bestScore`.

## Controls

- **Keyboard:** Arrows or WASD to move, Space to fire, `X` or Shift to bomb, `P` or Esc to pause.
- **Mouse:** move to steer, click to fire.
- **Touch:** drag left side to fly (virtual joystick), tap FIRE, tap BOMB.

## Persistent keys (`localStorage`)

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; defaults to the OS `prefers-reduced-motion` setting if unset |

Reading and writing these keys can throw in some privacy modes; keep those calls defensive if you touch them.

## Conventions when editing

- Keep the game in one file unless the user asks for a split. Preserve the IIFE and `'use strict'`.
- Use the existing naming: `*El` for DOM references, `STORAGE_*` for storage keys, `UPPER_SNAKE` for constants, and `ENEMY_TYPES` / `BOSSES` / `DIFFICULTIES` as the data tables for content. Add new enemies, bosses, and themes as entries in those tables.
- Gameplay tuning belongs in the constants and tables, not hard-coded in logic.
- Respect the reduced-motion and quality settings when adding visual effects (`reducedMotion`, `quality`). Route sound through `sfx` so mute works.
- Keep layouts working at phone width and on touch, since the game supports touch controls and `touch-action: none`.
- Do not add tracking, network calls, or third-party assets.

## Git

- Development branch for this work: `claude/gifted-tesla-tiey5u`.
- Commit with clear messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
