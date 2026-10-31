# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game (HTML, CSS, and JavaScript) lives in a single file, `index.html` (~1,670 lines). `README.md` is a placeholder.

There is no build step, package manager, test suite, linter, or CI configuration. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install is needed.

## File layout of `index.html`

- **`<head>`**: meta tags (viewport disables user scaling), an inline SVG favicon, and the page `<title>`.
- **`<style>`** (around lines 10–359): all CSS. Includes the HUD, the stat pills, hearts and bombs, the damage vignette, and the boss bar. Animations are defined here (for example `heartBeat`).
- **`<body>`** (around lines 361–421): the `#gameCanvas` element, HUD markup (`#score`, `#best`, `#mult`, `#weaponStat`, `#hearts`, `#bombs`), touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and the start/pause overlay (`#overlay`, `#overlayCard`, `#startBtn`, difficulty buttons `.diffBtn`).
- **`<script>`** (starts around line 422): one IIFE containing all game logic. Sections, roughly in file order:
  - DOM references and `STORAGE_*` keys.
  - Config tables: `DIFFICULTIES`, `THEMES` (background palettes per stage), `ENEMY_TYPES`, `BOSSES`.
  - Audio: a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and the `sfx` map.
  - Rendering helpers: `drawPlane`, `drawBossShip`, hearts/bombs HUD rendering, flash, screen shake, and combo/stage banners.
  - Input: keyboard (`keys`), touch joystick, touch and mouse fire buttons, and mouse steering.
  - Game state (`player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, `boss`, and so on), plus `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, and `applyPowerUp`.
  - Weapons and boss attack patterns: `fireWeapon`, `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, and the `BOSS_PATTERNS` array.
  - `update(dt)` (the simulation step), `draw()`, `endGame`, `togglePause`, `loop(now)` (the `requestAnimationFrame` driver), and the start-button handler.

Use `grep -n` for function names rather than relying on the line numbers above, since they will drift as the file changes.

## Game mechanics reference

- **Canvas**: fixed logical size 480×720 (`BASE_W`, `BASE_H`). `resize()` scales the displayed size to fit the window, capped at 1.6×.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rates, enemy speed, enemy shoot chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages**: after `KILLS_PER_STAGE` (18) stage kills, a boss is spawned (about 1.2s later). Defeating the boss (`killBoss`) increments `stage`. Stage themes cycle through `THEMES`. Bosses cycle through `BOSSES` as `(stage - 1) % BOSSES.length`, and boss HP is `70 + stage * 35`, scaled by the boss and difficulty multipliers.
- **Enemies**: `grunt` (stage 1+), `interceptor` (weaving, stage 1+), `tank` (4 HP, stage 2+).
- **Power-ups** (`maybeSpawnPowerUp`): `life`, `bomb`, and `weapon`. A weapon power-up raises the weapon level up to 3, then toggles between `vulcan` and `laser` and resets to level 1.
- **Scoring**: kills add score with a chain/combo multiplier (`chain`, `chainTimer`, `multiplier`). Taking damage breaks the chain.
- **Bombs**: `useBomb()` clears all enemy bullets, destroys all regular enemies (5 points each), and deals 24 damage to the boss. Bomb count is capped at `maxBombs`.

## Controls

- **Move**: arrow keys or WASD, mouse hover (plane glides toward pointer), or touch joystick.
- **Fire**: hold `Space`, or hold the on-screen fire button (touch or mouse).
- **Bomb**: `X` or `Shift`, or the on-screen bomb button.
- **Pause**: `P` or `Escape`.

## Persistence

The game uses `localStorage` with these keys. Reads and writes are direct (no try/catch wrapper); the only try/catch in the file guards audio setup. Keep that in mind if you change them.

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score |
| `skyfighter.muted` | Audio mute flag (`'1'` or `'0'`) |
| `skyfighter.difficulty` | Selected difficulty name |
| `skyfighter.reducedMotion` | Reduced-motion override (`'1'` or `'0'`). If unset, the `prefers-reduced-motion` media query is used. |

Changing any key name breaks existing players' saved data, so only do so deliberately.

## Conventions

- **Keep it a single file.** New game code, styles, and assets should go into `index.html` unless the user asks for a split. Do not introduce a bundler, framework, or external dependencies without asking.
- **No external network calls.** The game is self-contained. The only data URI is the favicon.
- **Preserve performance tuning.** The `quality` level (0 full, 1 medium, 2 low) is driven by `trackPerf` and scales particle counts. Keep new effects respecting it.
- **Respect reduced motion and mute.** New visual effects should check `reducedMotion`, and new sounds should go through `sfx`/`beep`/`noiseBurst` so they honor `muted`.
- **Accessibility.** Announcements for screen readers go through `announce()` (`#srLive`).
- **Mobile.** Keep the layout working at phone width and with touch controls. The page uses `touch-action: none` and `user-select: none` on purpose.

## Verifying changes

Since there is no test suite, verify changes by:

1. Opening `index.html` in a browser (or using the pre-installed Playwright/Chromium headless) and confirming there are no console errors on load.
2. Starting a game, playing through a stage transition and a boss fight, and checking the HUD, score, and pause/resume behavior.
3. Checking the layout at a narrow (phone) viewport.

Mention in your summary which of these you actually ran.

## Git

- Development branch for this work: `claude/gifted-tesla-g6i9xa`.
- Write clear, imperative commit messages describing the change.
- Do not open pull requests unless explicitly asked.
