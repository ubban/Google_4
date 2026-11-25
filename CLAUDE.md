# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-file HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`). About 1,670 lines.
- `README.md` — minimal project readme.

There is no build step, package manager, bundler, or test suite. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static file server. Mobile touch controls appear automatically on touch devices.

## Code layout (`index.html`)

- **Markup/CSS (lines ~1–420):** HUD pills, hearts/bombs display, overlay card, boss health bar, touch controls, and theme/entrance animations.
- **Script (starts ~line 422):** one IIFE wrapped in `'use strict'`. Nothing is exposed on `window`; keep it that way unless there is a reason.

Major sections inside the script, roughly in file order:

| Area | Key identifiers |
| --- | --- |
| DOM refs | `canvas`, `ctx`, `overlay`, `scoreEl`, `heartsEl`, `bombsEl`, … |
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` |
| Adaptive quality | `quality` (0 full / 1 medium / 2 low), `trackPerf()` |
| Tuning data | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `panFor()`, `EXPLODE_SFX` |
| Layout/input | `BASE_W = 480`, `BASE_H = 720`, `resize()`, joystick and pointer handlers |
| Game state | `resetGame()`, `update(dt)`, `draw()`, `loop(now)` |
| Spawning/combat | `spawnEnemy()`, `spawnBoss()`, `fireWeapon()`, `useBomb()`, `applyPowerUp()` |
| Boss attacks | `BOSS_PATTERNS` array and `fire*` pattern helpers, `updateBoss()`, `killBoss()` |
| Flow | `loseLife()`, `endGame()`, `togglePause()` |

The game loop is driven by `requestAnimationFrame`. `dt` is clamped to 50 ms, so frames that stall do not cause large jumps.

## Game design constants

When changing balance, edit the data tables rather than hard-coding numbers in logic:

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal`, `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting `lives`, starting `bombs`, and `bossHpMul`.
- **Stages:** a stage advances every `KILLS_PER_STAGE` (18) kills. Stage-gated enemies use `minStage`.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaving), `tank` (4 HP).
- **Bosses** (`BOSSES`): six bosses, cycled in order. Each has its own colors and `hpMul`. Attack patterns live in `BOSS_PATTERNS`.
- **Themes** (`THEMES`): six sky palettes that cycle with stage.

## Persistence

All state is stored in `localStorage` under the `skyfighter.*` prefix:

| Key | Value |
| --- | --- |
| `skyfighter.bestScore` | high score (number) |
| `skyfighter.muted` | `'1'` or `'0'` |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; if unset, falls back to `prefers-reduced-motion` |

Keep these keys stable. Renaming one silently resets that user's saved value.

## Conventions

- **Single file.** Add new CSS, JS, and markup to `index.html` unless the user asks for a split. Do not introduce a build step or external dependencies without asking.
- **No external network calls.** The game is fully self-contained.
- **Accessibility:** announcements go through `announce()` (the `srLive` live region). Respect `reducedMotion`, which disables or tones down screen shake, flashes, and similar effects. Keep the `prefers-reduced-motion` fallback.
- **Audio** is generated with the Web Audio API and starts only after a user gesture (`ensureAudio()`). Do not add audio files without asking.
- **Input** must work for keyboard, mouse/pointer, and touch. Touch handlers use `{ passive: false }` where they call `preventDefault()`.
- Keep DOM lookups at the top of the script, and keep tuning values in the constant tables.
- Match the existing style: 2-space indent, single quotes, `const`/`let`, and small named functions.

## Verification

There are no automated tests. After changing game logic, load `index.html` in a browser and check the affected path: start, fire, bomb, pause, game over, difficulty change, mute, and touch controls where relevant. For visual changes, check both desktop and a narrow phone-sized viewport.

## Git

- Development branch for this work: `claude/gifted-tesla-ldjoxc`.
- Commit messages should describe the gameplay or code change, not the tooling used.
