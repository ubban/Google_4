# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (`<style>`, starts ~line 10), and JavaScript (`<script>`, starts ~line 422). No external assets, no build step.
- `README.md` — minimal placeholder.

There is no `package.json`, bundler, test suite, linter, or CI config in the repo. Verification is manual: open `index.html` in a browser.

## Running the game

```bash
# Any static server works; or just open the file directly
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code structure (`index.html`)

The script is one IIFE (`'use strict'`) with these sections, roughly in file order:

| Area | What it holds |
| --- | --- |
| DOM refs | `const xxxEl = document.getElementById(...)` block at the top of the script |
| Storage keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*` in `localStorage`) |
| Adaptive quality | `trackPerf()` drops/raises `quality` (0–2) based on rolling frame time |
| Tuning tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Audio | Lazy `AudioContext` via `ensureAudio()`; `beep()` / `noiseBurst()` synth helpers; `sfx` table; `EXPLODE_SFX` |
| UI helpers | `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (aria-live) |
| Layout / input | `resize()` scales a `BASE_W × BASE_H` (480×720) logical canvas; keyboard (`keys[e.code]`), mouse, and touch joystick / FIRE / BOMB handlers |
| Game state | `resetGame()`; player, bullets, enemies, bosses, power-ups, explosions |
| Spawning & combat | `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `addScore`/`breakChain` (kill-chain multiplier), `applyPowerUp` |
| Boss AI | `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, seeded homing, double ring) driven by `updateBoss()` |
| Lifecycle | `loseLife()`, `killBoss()`, `endGame()`, `togglePause()` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Main loop | `loop(now)` — clamps `dt` to 50 ms, calls `update` then `draw` when running and not paused |

### Key gameplay constants

- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy shoot chance, lives, starting bombs, and boss HP multiplier.
- `KILLS_PER_STAGE = 18`; a boss spawns ~1.2 s after a stage's kill quota is met. Stage themes cycle through `THEMES`.
- Weapons: `vulcan` (spread grows with level 1–3) and `laser`. Power-ups grant weapon level, bombs, or an extra life (capped at `maxLives` / `maxBombs`).

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not add a build system or dependencies without a request.
- **No frameworks.** Vanilla JS and Canvas 2D only.
- **Tuning lives in tables.** Prefer changing values in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, and `BOSS_PATTERNS` over hard-coding numbers in logic.
- **Persistence** goes through the `skyfighter.*` `localStorage` keys. Wrap new storage access so the game still works if storage is blocked (e.g. private mode).
- **Accessibility:** respect the reduced-motion toggle (`reducedMotion`, `body.reduced-motion`) for any new shake, flash, or pulse effect; announce important state changes via `announce()`.
- **Mobile:** new controls must work with touch (`touchstart` with `preventDefault` where needed) as well as mouse and keyboard. The viewport is locked (`user-scalable=no`).
- **Audio** only starts after a user gesture — call `ensureAudio()` from input handlers rather than at load.
- Keep the game loop allocation-light; the adaptive quality system exists because frame time matters.

## Verifying changes

Since there are no automated tests:

1. Load `index.html` in a desktop browser and a mobile-sized viewport.
2. Check Start → play → pause/resume → game over → Play Again.
3. Exercise any touched system (weapons, bombs, boss patterns, difficulty switch, mute, reduced-motion toggle).
4. Confirm the browser console shows no errors.

## Git workflow

- Develop on the branch assigned for the session; do not push to `main` without explicit permission.
- Write descriptive commit messages that explain the gameplay or UI change.
- Do not open a pull request unless explicitly asked.
