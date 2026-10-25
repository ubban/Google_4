# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- Everything lives in `index.html`: markup, CSS (in a `<style>` block), and JavaScript (one IIFE in a `<script>` block).
- There is no build step, no bundler, no `package.json`, and no test suite. Open the file in a browser to run it.
- `README.md` is a placeholder: a title line and a stray timestamp. It has no project documentation yet.

## Working in this repo

- **Run it:** open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install is needed.
- **Keep it single-file.** New code goes into the existing `index.html` unless the user asks for a split. Do not add dependencies, a bundler, or external CDN scripts without asking.
- **Match the existing style.** The JS uses `const`/`let`, arrow functions, section comments like `// ---------- Name ----------`, and short helper functions. CSS uses custom properties and small, named selectors. Follow the surrounding code's naming and comment density.
- **Verify in a browser.** There are no automated tests. For gameplay or UI changes, load the page and exercise the affected path (start, fire, bomb, pause, boss, game over, touch layout). Say so explicitly if you could not do this.
- **Git:** develop on the branch the session names. Commit with clear messages and push with `git push -u origin <branch>`.

## Architecture of `index.html`

### Markup
- `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`, `#topBar`, `#hearts`, `#bombs`, `#weaponName`, `#weaponLvl`, `#mult`), the boss bar (`#bossBarWrap`), transient popups (`#comboPop`, `#stageBanner`), touch controls (`#touchControls`), and the start/pause/game-over card (`#overlay`, `#overlayCard`).
- `#srLive` is a polite ARIA live region. Use the `announce(text)` helper to update it rather than writing to it directly.

### Script structure
All game logic is inside one IIFE with `'use strict'`. Major sections, in source order:

1. **DOM references and constants:** element lookups, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (background palettes cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six boss designs).
2. **Settings and persistence:** best score, mute, difficulty, and reduced-motion preference are read from and written to `localStorage` under the `skyfighter.*` keys.
3. **Adaptive performance:** `trackPerf` samples frame time and steps a `quality` level (0–2) up or down.
4. **Audio:** a Web Audio context created lazily by `ensureAudio()` (call it from a user gesture). `beep`, `noiseBurst`, and the `sfx` object produce all sounds. There are no audio files.
5. **HUD and effects:** `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
6. **Input:** keyboard (`keys` map, keyed by `KeyboardEvent.code`), mouse on the canvas, and touch joystick plus fire/bomb buttons.
7. **Game state and flow:** `resetGame`, `update(dt)`, `draw()`, `loop(now)`, `togglePause`, `endGame`, `loseLife`.
8. **Combat:** `fireWeapon`, `spawnEnemy`, `spawnBoss`, `useBomb`, `addScore` / `breakChain` (kill-chain multiplier), `maybeSpawnPowerUp`, `applyPowerUp`.
9. **Bosses:** `BOSS_PATTERNS` is an array per boss of attack functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss` runs them; `killBoss` advances the stage.
10. **Rendering helpers:** `drawPlane`, `drawBossShip`, `drawClouds`, plus the main `draw()`.

### Game rules to keep in mind
- **Difficulty** (`DIFFICULTIES`) sets lives, bombs, spawn rate, enemy speed, enemy fire chance, and a boss HP multiplier. Changing one field changes balance, so check the others.
- **Stages:** the player must defeat `KILLS_PER_STAGE` (18) enemies to trigger a boss. The boss is spawned about 1.2 s later. Defeating a boss increments `stage`, and the boss list and theme cycle by `(stage - 1)`.
- **Weapons:** `vulcan` (spread shot) and `laser` (piercing lanes), each with power levels 1–3. Power-up types are `weapon`, `bomb`, and `life`.
- **Controls:** arrows or WASD to move; Space or hold-to-fire on touch to shoot; `KeyX` or Shift to bomb; `KeyP` or Escape to pause.
- **Persistence:** only the best score and settings are saved. Run state is not.
- **Accessibility:** respect `prefers-reduced-motion` (the `reducedMotion` flag toggles the `reduced-motion` body class). Keep the live region and keyboard controls working when you change UI.

## Conventions
- Keep gameplay constants in the existing config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) rather than as magic numbers scattered through the update loop.
- When adding a new enemy, boss, or power-up, update every place that switches on its type (spawn, update, draw, and any HUD or label mapping).
- Keep touch and keyboard input paths in sync when adding a new action.
- Commits in this repo start with a short imperative summary line, as in `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.
