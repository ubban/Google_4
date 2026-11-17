# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire app lives in one file, `index.html` (inline `<style>` and one inline `<script>` in an IIFE). There is no build step, no package manager, no bundler, no test suite, and no external dependencies. `README.md` is a placeholder.

To run it, open `index.html` in a browser (or serve the directory with any static server). No install step is needed.

## Repository layout

- `index.html` — the whole game: markup, CSS, and JavaScript.
- `README.md` — minimal placeholder.
- `CLAUDE.md` — this file.

## Code structure of `index.html`

The inline script is one strict-mode IIFE. Major sections, roughly in file order:

1. **DOM references** — `const` lookups for canvas, HUD elements, overlay, touch controls, and buttons.
2. **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.` in `localStorage`. Every read should tolerate a missing or invalid value.
3. **Tuning tables** — `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses with colors and HP multipliers). Balance changes should go here first.
4. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, `panFor()`. Audio is built on the Web Audio API and is created lazily from a user gesture (`ensureAudio()` is called on start, fire, and bomb input).
5. **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
6. **Input** — `keydown`/`keyup` on `window`, joystick and fire/bomb buttons for touch, mouse and canvas pointer handling.
7. **Game state and lifecycle** — `resetGame()`, `togglePause()`, `setDifficulty()`, `resize()` (fits a 480×720 base canvas, `BASE_W`/`BASE_H`).
8. **Rendering** — `drawPlane`, `drawBossShip`, plus `draw()`.
9. **Spawning and entities** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `rectsOverlap` (AABB collision).
10. **Scoring and combat** — `addScore`, `breakChain` (kill-chain multiplier), `useBomb` (clears screen, damages bosses), `fireWeapon` (Vulcan/Laser with three power levels), `applyPowerUp`.
11. **Boss patterns** — `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, collected in `BOSS_PATTERNS`; `updateBoss(dt)` drives them.
12. **Main loop** — `update(dt)` and `draw()` called from `loop(now)` via `requestAnimationFrame`. `dt` is clamped to 50 ms. Updates are skipped while `!running || paused`. `trackPerf(dt)` runs every frame.

## Game mechanics (for orientation)

- Stages advance after `KILLS_PER_STAGE` kills; each stage ends with a boss fight with a health bar.
- Lives are hearts; bombs are limited per difficulty and clear the screen.
- Power-ups include weapon level, bomb, and extra life.
- Controls: arrow keys or WASD to move, Space or held fire to shoot, X or Shift to bomb, P or Escape to pause. Touch uses a joystick, fire button, and bomb button. Mouse drives the plane via `mousemove` on the canvas.
- Accessibility: reduced-motion defaults from `prefers-reduced-motion` and can be toggled by the user (persisted). When on, `screenShake()` caps shake duration and the `body.reduced-motion` class in CSS disables the vignette pulse, heart pulse, combo pop, stage banner, and overlay card animations.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a build tool, framework, or CDN dependency without asking.
- Keep the script inside the existing IIFE and `'use strict'`. Do not add new globals.
- Use `const`/`let`, arrow functions, and the existing naming style (camelCase functions, UPPER_CASE tuning constants).
- Game logic should use `dt` (seconds) so behaviour is frame-rate independent.
- Wrap `localStorage` access so the game still runs when storage is unavailable (private mode, blocked storage).
- Preserve touch support when changing input: any new control needs both keyboard and touch paths, and `touch-action`/`preventDefault` settings should keep the page from scrolling.
- Preserve the mobile layout: the canvas scales to the viewport and the page should not scroll horizontally.

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser (desktop and, if the change touches layout or input, a mobile-sized viewport).
2. Play through start, pause/resume, a bomb, a boss fight, and game over; confirm the best score persists across reloads.
3. Check the browser console for errors.
4. Toggle mute and reduced-motion and confirm both persist.

Headless browser checks (e.g. Playwright with Chromium) are a reasonable way to automate step 2 if a test harness is added later.

## Git workflow

- Default working branch for automated work in this environment: `claude/gifted-tesla-olg83h`.
- Commit with clear, imperative messages describing the gameplay or code change.
- Do not force-push. Do not open pull requests unless the user asks.
