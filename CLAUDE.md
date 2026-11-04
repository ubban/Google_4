# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (an IIFE, `'use strict'`). No build step, no bundler, no dependencies.
- `README.md` — placeholder only.

There is no package.json, test suite, linter, or CI config. Verify changes by opening `index.html` in a browser (or by serving the directory with any static server, e.g. `python3 -m http.server`).

## Working on the code

- Keep the game as a single self-contained file unless the user asks for a split. Do not introduce a framework or build tooling without being asked.
- Prefer small, targeted edits with the Edit tool; the file is ~58 KB, so read the relevant section (use line ranges) rather than the whole file.
- Constants live at the top of the script section, roughly lines 460–520: storage keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`. Tune gameplay there first.
- Test every change in a real browser for both desktop and touch layouts. The layout must work at phone width (viewport meta disables zoom; `touch-action: none` is set on the page).

## Architecture map (`index.html`)

Markup section (~lines 360–420):
- `#gameCanvas` — the only drawing surface, sized in `resize()`.
- `#hud` — pill-style stats: score, best, multiplier, weapon, hearts (lives), bombs.
- `#overlay` — title / pause / game-over card with `#startBtn` and difficulty buttons (`.diffBtn`).
- `#touchControls` — joystick, `#fireBtn`, `#bombBtn`, shown on touch devices.
- `#vignette`, `#flash`, `#stageBanner`, `#comboPop`, `#bossBarWrap` — transient visual feedback.
- `#srLive` — an `aria-live` region; game events are announced via `announce(text)`. Keep it updated when adding new player-facing events.

Script section (~lines 422 onward), grouped roughly as:
1. **DOM refs and storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `localStorage`, prefixed `skyfighter.`). Reads/writes are not wrapped in try/catch; keep that in mind if you touch them.
2. **Adaptive quality** — `trackPerf(dt)` samples frame times and steps `quality` 0 (full) → 2 (low) when FPS drops; draw code checks it to reduce effects.
3. **Audio** — a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, `sfx` table). Audio starts only after a user gesture via `ensureAudio()`. Mute state is persisted.
4. **Difficulty** — `DIFFICULTIES` (easy / normal / hard) controls spawn rate, enemy speed, shoot chance, starting lives and bombs, and boss HP multiplier. `setDifficulty(name)` persists the choice.
5. **Input** — `keys` map from `keydown`/`keyup` (`e.code`), mouse steering and click-to-fire on the canvas, and touch joystick/buttons. Desktop bindings: arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
6. **Game state** — `resetGame()` initialises `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `stage`, `stageKills`, `frame`, `kills`.
7. **Entities and spawning** — `spawnEnemy()` (types filtered by `minStage`), `spawnBoss()`, `maybeSpawnPowerUp()` (types: `life`, `bomb`, `weapon`), `spawnExplosion()`.
8. **Weapons and bombs** — `fireWeapon()` implements Vulcan and Laser at up to 3 power levels; `useBomb()` clears the screen and damages bosses; `applyPowerUp(type)` applies pickups.
9. **Boss patterns** — `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss(dt)`. `killBoss()` advances the stage.
10. **Scoring** — `addScore(base)` applies the kill-chain multiplier; `breakChain()` resets it when the player is hit.
11. **Flow** — `loseLife()` → `endGame()`; `togglePause()`; stage progression after `KILLS_PER_STAGE` kills triggers a boss, then the next stage. Stage themes cycle through `THEMES` by `(stage - 1) % THEMES.length`.
12. **Rendering** — `draw()` renders background (`drawClouds`, sun glow, theme gradient), `drawPlane`, `drawBossShip`, bullets, power-ups, particles, HUD sync (`renderHearts`, `renderBombs`, etc.), and the low-health vignette.
13. **Main loop** — `update(dt)` then `draw()` inside `loop(now)`, driven by `requestAnimationFrame`. Keep update logic out of `draw()`.

## Conventions

- Game logic should run from `update(dt)` with time-based movement; `draw()` must stay side-effect free apart from canvas and DOM presentation.
- Respect the existing reduced-motion setting (`body.reduced-motion`, `skyfighter.reducedMotion`) when adding screen shake, flashes, or animations.
- Keep new persistent state under the `skyfighter.` key prefix.
- Keep the accessibility hooks working: announce major events through `announce()`, and keep controls reachable by keyboard and touch.
- Don't add external scripts, fonts, or network calls; the game currently loads nothing remote except its inline data-URI favicon.

## Git

- Development branch for this work: `claude/gifted-tesla-t0tuy7`. Commit and push there; do not push to other branches without permission.
- Write clear commit messages describing the gameplay or UI change.
