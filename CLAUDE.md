# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a tiny static web project containing one game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — a placeholder with only the project title.

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not add any of these unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Code layout inside `index.html`

- **`<style>` (top of file):** dark color scheme, HUD positioning (`#hud`, `#weaponStat`, `#bossBarWrap`), overlay and mobile control styles. Uses safe-area insets for notched phones.
- **`<body>` markup (~line 363 onward):** the `#gameCanvas` element, HUD elements, start/pause/game-over overlay (`#overlay`), and touch controls (move zone, FIRE, BOMB).
- **Script (starts ~line 422):** a single IIFE holding all game logic. Major sections, in file order:
  - Constants: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - Settings and persistence: best score, mute, difficulty, reduced-motion (all in `localStorage`).
  - Audio: lazily created `AudioContext`; `beep()` and `noiseBurst()` synthesize every sound effect (no audio files).
  - Layout: `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport.
  - Input: keyboard (`keys[e.code]`), mouse aim/fire, touch joystick and buttons.
  - Game state: `resetGame()`, `update(dt)`, `draw()`, `loop(now)` driven by `requestAnimationFrame`. `dt` is clamped to 0.05 s.
  - Combat: `spawnEnemy`, `fireWeapon`, `useBomb`, `applyPowerUp`, `spawnBoss`, `updateBoss`, `killBoss`, and the boss bullet patterns in `BOSS_PATTERNS` (`fireRadial`, `fireCross`, `fireHomingSeeded`, etc.).
  - Lifecycle: `loseLife()`, `endGame()`, `togglePause()`.
  - Adaptive quality: `trackPerf()` lowers the `quality` level (0 full, 1 medium, 2 low) when frame time is poor.

## Game rules worth knowing

- Three difficulty presets (`easy`, `normal`, `hard`) change spawn rate, enemy speed, shooting chance, starting lives, starting bombs, and boss HP multiplier. Change balance in `DIFFICULTIES`.
- Each stage ends after `KILLS_PER_STAGE` (18) kills, which spawns that stage's boss. Boss = `BOSSES[(stage - 1) % BOSSES.length]`; background theme = `THEMES[(stage - 1) % THEMES.length]`. The stage number increments only when the boss is defeated (`killBoss()`).
- Weapons are `vulcan` and `laser`. Each power-up raises the weapon level up to 3; a power-up at level 3 switches to the other weapon and resets the level to 1 (`applyPowerUp`).
- Smart bombs: `X` or `Shift`, with `bombCount` / `maxBombs`.

## Controls (keep the in-game hint in sync)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag left side to fly, FIRE and BOMB buttons.

The hint text lives in `#controlsHint` (~line 417). If you change a binding, update that text too.

## Persistence

Settings use `localStorage` with the keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Keep these names stable; changing them resets players' saved data. All reads should tolerate missing or invalid values, as the current code does.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce external dependencies or CDN scripts.
- Preserve the single-IIFE structure and the `const`/`let` style already in use.
- Preserve the 480 × 720 logical resolution. Gameplay numbers are tuned against it.
- Keep the page usable at phone width and with touch input; the touch handlers use `passive: false` where `preventDefault` is needed.
- Sound effects are synthesized with Web Audio. Do not add audio files without a reason.
- Prefer small, targeted edits with the existing helper functions (`beep`, `spawnExplosion`, `screenShake`, `announce`) over new abstractions.

## Verifying changes

There is no automated test or lint setup. After a change, open `index.html` in a browser (or use a headless browser) and check at least:

1. The game starts from the overlay and runs without console errors.
2. Movement, firing, bombs, and pause all respond.
3. A stage can be reached or simulated, and a boss appears and can be defeated.
4. The game-over flow shows the score and updates the best score.
5. Touch controls work at a phone-sized viewport, if the change touches input or layout.

## Git

- Development branch for this session: `claude/gifted-tesla-5sowe9`.
- Commit messages should be short and describe the gameplay or file change.
