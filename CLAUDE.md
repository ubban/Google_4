# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up.

- `index.html` contains the entire game: markup, CSS (`<style>`, roughly lines 10–360), and one inline IIFE `<script>` (roughly lines 422–1668). There are no external dependencies, no bundler, and no package manager.
- `README.md` is a stub. It has no setup or usage instructions.
- Git history: `Initial commit`, then `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- **Build:** none. Edit the file and reload.
- **Tests / lint:** none exist. Verify changes by playing the game in a browser. A headless Playwright or Chromium run is an option for scripted checks.

## Code structure (`index.html`)

All game logic lives in one IIFE with `'use strict'`. Sections appear in roughly this order:

| Section | Key identifiers | Purpose |
|---|---|---|
| DOM refs | `canvas`, `overlay`, `scoreEl`, … | Cached element lookups |
| Storage keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` | `localStorage` keys prefixed `skyfighter.` |
| Perf | `trackPerf`, `quality` (0 full / 1 medium / 2 low) | Adaptive quality based on the rolling average frame time |
| Config tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` | Balance and content data |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx.*`, `EXPLODE_SFX` | Web Audio synthesized sound effects, no audio files |
| HUD / FX | `renderHearts`, `renderBombs`, `showStageBanner`, `screenShake`, `flashHit` | DOM HUD and visual effects |
| Layout | `BASE_W = 480`, `BASE_H = 720`, `resize` | Fixed logical canvas size, scaled to fit the viewport |
| Input | `keys`, joystick helpers, `canvasPointFromEvent` | Keyboard, mouse, and touch handling |
| Game state | `resetGame`, `update(dt)`, `draw()` | Main simulation and render |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp` | Entity creation |
| Boss patterns | `BOSS_PATTERNS`, `fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`, … | Bullet-hell attack patterns |
| Lifecycle | `loseLife`, `killBoss`, `endGame`, `togglePause` | Game flow |
| Loop | `loop(now)` | `requestAnimationFrame` driver, `dt` clamped to 50 ms |

## Game mechanics to keep in mind

- **Difficulty** (`DIFFICULTIES`) sets spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. The choice persists in `localStorage`.
- **Stages:** every `KILLS_PER_STAGE` (18) kills triggers a boss. Defeating a boss increments `stage`. The background theme cycles through `THEMES` by `(stage - 1) % THEMES.length`.
- **Enemies:** `grunt`, `interceptor` (fast, weaving), and `tank` (4 HP). Each type has a `minStage` gate.
- **Bosses:** `BOSSES` supplies names, colors, and `hpMul`. Each uses a pattern from `BOSS_PATTERNS`.
- **Scoring:** `addScore` applies a combo multiplier that `breakChain` resets, and taking damage also breaks the chain.
- **Best score** is stored under `skyfighter.bestScore`.

## Controls

- **Keyboard:** Arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- **Mouse:** move to steer, click to fire.
- **Touch:** drag on the left zone to fly, FIRE button, BOMB button. Touch controls appear only on touch devices.

## Conventions

- Keep everything in `index.html` unless the user asks for a restructure. Splitting into modules would change how the project loads.
- Keep the code in the existing style: `const`/`let`, arrow functions in callbacks, section comments with `// ---------- Name ----------`, and tuning values in the config tables rather than hard-coded in logic.
- Use Web Audio for any new sound effects, following the `beep` and `noiseBurst` helpers. Do not add audio files.
- Respect `reducedMotion`. Any new screen shake, flash, or particle effect should check it, as the existing effects do.
- Keep UI text and `announce()` calls in sync for screen-reader updates when game state changes.
- Gameplay changes should stay within the fixed `480x720` logical coordinate space.
- Preserve `localStorage` keys. Renaming them would reset players' saved best score, mute, difficulty, and motion settings.
- Wrap new `localStorage` access in try/catch if the code will run in private or restricted browsing contexts. The current code does not, so match or improve on that deliberately.

## Git

- Development branch for this task: `claude/gifted-tesla-w0von7`.
- Commit messages should be short and imperative, and describe the gameplay or code change.
