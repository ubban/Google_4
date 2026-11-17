# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 canvas app.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines).
- `README.md` — placeholder; currently only a title line.

There is no build step, no package manager, no bundler, and no test suite. Do not introduce one unless asked.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify changes by playing them in a browser, since there are no automated tests.

## Code layout inside `index.html`

- **Markup**: canvas (`#gameCanvas`), HUD (score, best, multiplier, weapon, hearts, bombs), overlay card (title, difficulty buttons, start button), touch controls (joystick, fire, bomb), and pause/mute/motion buttons.
- **CSS**: a single `<style>` block. HUD is pill-style; overlay is a glassmorphic card.
- **JavaScript**: one IIFE (`'use strict'`) containing all game logic. Major sections, in rough order:
  - DOM references and constants (`STORAGE_*`, `DIFFICULTIES`).
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is created lazily on first user gesture.
  - HUD/UI helpers: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
  - Input: keyboard (`keys`), touch joystick and buttons, mouse on canvas.
  - Game state: `resetGame`, `update(dt)`, `draw()`, `loop(now)`.
  - Entities: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `applyPowerUp`.
  - Weapons and boss patterns: `fireWeapon`, `useBomb`, `updateBoss`, `killBoss`, plus the `fire*` pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring).
  - Flow: `loseLife`, `endGame`, `togglePause`, `startBtn` handler.

Keep changes inside this structure. If you add a feature, follow the existing naming and section placement rather than creating a parallel mechanism.

## Game configuration

Difficulty tuning lives in the `DIFFICULTIES` object (`easy`, `normal`, `hard`). Each entry sets:
`spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.

Spawn rate ramps with `Math.floor(frame / 300) * 4` in `update`. Change tuning values there rather than hardcoding numbers in logic.

## Persistence

All state is stored in `localStorage` under keys prefixed `skyfighter.`:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score |
| `skyfighter.muted` | Audio muted (`'1'` / `'0'`) |
| `skyfighter.difficulty` | Selected difficulty name (default `normal`) |
| `skyfighter.reducedMotion` | Reduced-motion preference (`'1'` / `'0'`; unset falls back to the OS preference) |

Renaming a key resets that user's saved value, so keep existing keys stable unless the change is intentional. Reads are not wrapped in try/catch, so storage failures (e.g., blocked storage) will throw; if you touch these paths, consider guarding them.

## Conventions

- Keep everything self-contained in `index.html`. Do not add external scripts, stylesheets, or fonts without a specific request.
- Preserve accessibility hooks: the `srLive` live region (`announce`), the reduced-motion toggle, and keyboard/touch parity for controls.
- Preserve mobile support: touch controls and the `resize` / `orientationchange` handling must keep working at phone width.
- Use `dt`-based movement in `update` so gameplay is frame-rate independent.
- Keep the code style consistent with the surrounding code (2-space indent, single quotes in JS, `const`/`let`).

## Git

- Development branch for this task: `claude/gifted-tesla-bq73kh`.
- Commit messages should be descriptive; match the existing style (imperative subject line, bullet body for multi-part changes).
- Do not open pull requests unless explicitly asked.
