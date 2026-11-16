# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The player flies a plane, switches between two weapon types, drops smart bombs, and fights a boss at the end of each stage.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — a stub with only the title.

There is no build step, no package manager, no bundler, no framework, and no test suite. External assets are none; all sprites are drawn with the Canvas 2D API and all sound is synthesized with the Web Audio API.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage and one boss fight when touching gameplay code.
3. Check both desktop (keyboard) and touch layouts when touching input or HUD code.

If you add tooling (tests, a linter, a build step), document the commands here.

## Code layout inside `index.html`

Everything JavaScript lives in one IIFE (`(function () { 'use strict'; ... })();`) inside the final `<script>` block. The rough order is:

| Section | Purpose |
|---|---|
| DOM lookups | `getElementById` references for canvas, HUD, overlays, and touch buttons |
| Constants | `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table |
| HUD helpers | `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake` |
| Input | keyboard handlers, touch joystick (`joystick*`), fire/bomb buttons, mouse on canvas |
| Game state | `resetGame`, the `player` object, and the `bullets`, `enemies`, `powerUps`, and `enemyBullets` arrays |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` |
| Combat | `fireWeapon`, `useBomb`, `addScore`, `breakChain`, `rectsOverlap`, `spawnExplosion` |
| Bosses | `BOSS_PATTERNS` (bullet patterns such as `fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`), `updateBoss`, `killBoss` |
| Main loop | `update(dt)` for logic, `draw()` for rendering, `loop(now)` driven by `requestAnimationFrame` |
| Lifecycle | `loseLife`, `endGame`, `togglePause`, and the start button handler |

### Key mechanics

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), and `hard` set spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss. Stages cycle through `THEMES` (6 sky palettes) and `BOSSES` (6 bosses) using modulo indexing.
- **Enemies**: `grunt`, `interceptor` (fast, weaving), and `tank` (4 HP, appears from stage 2).
- **Weapons**: `vulcan` (spread shot) and `laser` (piercing beams). Weapon power-ups raise the level from 1 to 3; past level 3 they switch the weapon type and reset the level to 1.
- **Power-ups**: `life` (6%), `bomb` (8%), and `weapon` (20%) drop chances from destroyed enemies.
- **Adaptive quality**: `trackPerf` samples the frame time over 45 frames and steps a `quality` level (0–2) down when frames are slow. Reduced-motion mode disables screen shake, flashes, and pulses.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (best score, mute, difficulty, reduced motion). Storage access is not wrapped in try/catch, so guard any new storage calls.
- **Accessibility**: an `aria-live` region (`#srLive`) is updated through `announce()`. Use it for significant state changes such as stage transitions and game over.

## Conventions

- Keep the game in a single file unless there is a clear reason to split it. Preserve the IIFE wrapper so nothing leaks onto `window`.
- Use `const` and `let`; the file runs in strict mode.
- Match the existing style: two-space indentation, single quotes in JS, and short section comments with `// ---------- Name ----------` banners.
- Tune gameplay through the constants at the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in logic.
- Add new HUD or touch controls with the existing patterns: a DOM element in the markup, styles in the `<style>` block, and listeners registered in the input section.
- Keep the layout working at phone widths; the page uses `touch-action: none` and fixed-position containers.

## Git

- Development branch for this work: `claude/gifted-tesla-p6kd7x`. Commit and push there.
- Do not create a pull request unless explicitly asked.
- Commit history is short (two commits on the base branch); keep commit messages imperative and descriptive.
