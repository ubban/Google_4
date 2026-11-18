# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free front-end project. Its only substantive content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`. The whole game is a single self-contained file, `index.html` (~1,670 lines: CSS in one `<style>` block, markup, and one inline IIFE `<script>`).

| File | Purpose |
| --- | --- |
| `index.html` | The complete game: markup, styles, audio, rendering, input, and game logic. |
| `README.md` | Placeholder title only. |
| `CLAUDE.md` | This file. |

There is no build step, package manager, test suite, linter, or bundler. Do not introduce one unless the user asks for it.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by loading the page and playing: check the title overlay, a full run, pause, game over, and the best-score update.
- For mobile-layout changes, check at phone width. The touch controls only appear on touch-capable layouts.

## Architecture of `index.html`

The script runs inside a single `(function () { 'use strict'; ... })()` closure, so nothing leaks to `window`. Keep new code inside that closure.

Major sections, in file order:

1. **DOM references**: `const x = document.getElementById(...)` for every HUD, overlay, and touch element. IDs in markup and script must match.
2. **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all in `localStorage`, all wrapped as plain reads/writes today).
3. **Adaptive quality**: `trackPerf()` samples frame times and steps `quality` (0 full, 1 medium, 2 low) to shed effects on slow devices.
4. **Tuning tables** (edit these to rebalance rather than hard-coding numbers elsewhere):
   - `DIFFICULTIES`: `easy` / `normal` / `hard` with spawn rates, enemy speed, shoot chance, starting `lives`, starting `bombs`, and `bossHpMul`.
   - `KILLS_PER_STAGE` (18): kills needed before a boss wave.
   - `THEMES`: six sky palettes cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+).
   - `BOSSES`: six bosses (`VANGUARD`, `BEHEMOTH`, `SPECTER`, `DREADNOUGHT`, `WRAITH`, `TITAN`), each with colors and `hpMul`.
5. **Audio**: a Web Audio synth. `ensureAudio()` lazily creates the context on first user gesture; `beep()`, `noiseBurst()`, and the `sfx` object generate all sounds procedurally. There are no audio files.
6. **HUD helpers**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader live region).
7. **Layout**: a fixed logical canvas of `BASE_W = 480` × `BASE_H = 720`, scaled to fit in `resize()` (capped at 1.6×).
8. **Input**: keyboard (`keys` map keyed by `KeyboardEvent.code`), a virtual joystick and FIRE/BOMB buttons for touch, and mouse steering/click-to-fire on the canvas.
9. **Game state and loop**: `resetGame()` initializes state; `update(dt)` advances the simulation; `draw()` renders; `loop(now)` drives both via `requestAnimationFrame`.
10. **Weapons and power-ups**: `fireWeapon()` (Vulcan/Laser, three levels), `applyPowerUp()`, `maybeSpawnPowerUp()`.
11. **Bosses**: `spawnBoss()`, `updateBoss()`, `killBoss()`, and the `BOSS_PATTERNS` array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
12. **Lifecycle**: `loseLife()`, `endGame()`, `togglePause()`, and the start button handler at the bottom.

### Game-state conventions

- Score uses a kill-chain multiplier: `addScore()` applies it, `breakChain()` resets it.
- Bombs are limited per run and damage bosses (`useBomb()`).
- The best score is written to `localStorage` only at game end (`endGame()`).
- Reduced motion (`body.reduced-motion` plus the `reducedMotion` flag) softens screen shake (smaller and shorter), skips the hit/bomb flash, and turns off the low-health vignette pulse and several entrance animations via CSS. It defaults to the OS `prefers-reduced-motion` setting until the player toggles it (`motionBtn`).

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add npm dependencies or a build step.
- **Plain JavaScript, ES2017+ syntax already in use** (`const`/`let`, arrow functions, template strings). Match the existing style: two-space indent, single quotes in JS, section comments of the form `// ---------- Name ----------`.
- **Keep the closure intact.** Do not expose globals; add new functions inside the IIFE.
- **Change tuning in tables, not in logic**, when the change is a balance tweak.
- **Respect accessibility hooks**: keep `aria-live` announcements (`announce()`) for state changes that matter to screen-reader users, and keep the reduced-motion path working for any new visual effect.
- **Preserve storage keys.** Renaming a `skyfighter.*` key silently resets players' saved best score, mute, difficulty, and motion settings.
- **Mobile first.** Any UI change must work with touch controls and at phone widths without horizontal scrolling.

## Git and workflow

- Default working branch for this session: `claude/gifted-tesla-8y1nqy`. Commit and push there; do not push to other branches without permission.
- Do not open a pull request unless the user asks for one.

## Notes for future edits

- The `<title>`, `<meta name="description">`, and `theme-color` are in the `<head>`; update them together if the game is renamed.
- `README.md` is minimal. If you add setup or play instructions, keep them in `README.md` and keep this file focused on guidance for Claude.
