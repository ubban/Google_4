# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (one inline `<script>` wrapped in an IIFE, with `'use strict'`).
- `README.md` — a one-line title. Its body is only a timestamp line and has no setup or usage docs.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Do not add any of these unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). It has no external dependencies: no CDN scripts, fonts, or network requests. The only external-looking reference is a `data:` URI favicon.

## Code layout inside `index.html`

The script is organized by `// ----------` section comments and top-level helper functions. Key areas, roughly in file order:

- **DOM references and storage keys** (`STORAGE_*`): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. These are read from `localStorage` at startup.
- **Adaptive performance** (`trackPerf`): tracks frame times and steps a `quality` level (0–2) up or down.
- **Config tables**: `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes, cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six designs, cycled by stage).
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `sfx`): Web Audio synthesis only. No audio files.
- **Rendering helpers**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, and the `draw*` functions (`drawPlane`, `drawBossShip`, `drawClouds`, and `draw`).
- **Layout and input**: `resize` (fits a 480×720 logical canvas, `BASE_W`/`BASE_H`), a virtual joystick and fire/bomb buttons for touch, mouse steering on the canvas, and keyboard handlers.
- **Game state**: `resetGame` initializes the state variables (`player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, `boss`, `score`, `lives`, `bombCount`, `stage`, `stageKills`, and so on).
- **Simulation**: `update(dt)` runs per frame. Its helpers are `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `updateBoss`, `loseLife`, `killBoss`, `endGame`, and `togglePause`.
- **Boss attack patterns**: `BOSS_PATTERNS` is an array of functions built from `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`.
- **Loop**: `loop(now)` is a `requestAnimationFrame` loop. It clamps `dt` to 0.05 s, then calls `update` and `draw` only when running and not paused.

## Gameplay facts that are easy to get wrong

- **Controls.** Keyboard: arrows or WASD-style movement via `keys[e.code]`, `Space` to fire, `X` or either `Shift` for a bomb, `P` or `Escape` to pause. Mouse: the plane glides toward the pointer over the canvas. Touch: joystick, fire button, and bomb button.
- **Stages.** Killing `KILLS_PER_STAGE` (18) enemies triggers a boss about 1.2 s later. Defeating the boss increments `stage`. Enemy types gate on `minStage` (tanks appear from stage 2).
- **Boss HP** is `round((70 + stage * 35) * difficulty.bossHpMul * design.hpMul)`, where `design` is the boss for the current stage.
- **Power-ups** drop from destroyed enemies: life (6%), bomb (8%), and weapon (20%). Weapon pickups upgrade the level from 1 to 3, then toggle between `vulcan` and `laser` and reset to level 1.
- **Lives and bombs** depend on difficulty (`lives` 4/3/3, `bombs` 3/2/2).

## Conventions

- Keep the game in a single file unless the user asks for a split. Preserve the IIFE wrapper and the `'use strict'` mode.
- Match the existing style: two-space indentation, single quotes in JavaScript, `const` by default, and the `// ----------` section headers.
- Keep `localStorage` access tolerant of failure. Browsers can block it in private windows.
- Respect the reduced-motion setting (`reducedMotion`, `body.reduced-motion`) for any new animation or screen shake.
- Keep the layout working on phones: touch controls, `touch-action: none`, and the `viewport-fit=cover` safe-area handling already in place.
- Do not introduce external network requests, CDN assets, or third-party libraries without an explicit request.

## Verifying changes

There is no automated test. To verify a change:

1. Open `index.html` in a browser, or serve the directory statically.
2. Start a game, play through a stage transition, and defeat at least one boss if the change touches combat or bosses.
3. Check the HUD, the overlay, and the pause and game-over flows.
4. Check the layout at a narrow phone width. Touch controls only appear on touch-capable devices.

If the change is visual or interactive and a browser is available, a headless Playwright screenshot is a reasonable check. Save screenshots to the scratchpad directory, not the repository.

## Git

- Development branch for this session: `claude/gifted-tesla-wu6dwu`. Push there with `git push -u origin <branch>`.
- Do not force-push. Do not create pull requests unless explicitly asked.
- Commit messages should be short and imperative, like the existing `Add Sky Fighter: ...` history.

## Maintaining this file

Update this file when the structure, controls, tuning constants, or workflow change. Keep it factual. Do not add speculative sections, and do not record secrets, personal details, or environment-specific paths.
