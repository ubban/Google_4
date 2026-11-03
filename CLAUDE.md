# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- The entire app lives in **`index.html`** (~1,670 lines): inline CSS in `<style>`, markup in `<body>`, and one IIFE in a `<script>` block that contains all game logic.
- `README.md` is a placeholder and contains no setup or build instructions.
- There is **no build step, no package manager, no test suite, and no dependencies**. Nothing is bundled or transpiled.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- No automated tests exist. Verify changes by playing the game in a browser, ideally at both desktop and phone-sized viewports, since touch controls and the responsive layout are part of the feature set.
- Check the browser console for errors after any change to the script block.

## Code layout (inside `index.html`)

Markup, then script. The script is one IIFE with these sections, roughly in order:

| Area | What it holds |
|---|---|
| DOM refs | `getElementById` lookups for canvas, HUD, overlay, touch controls, buttons |
| Storage keys | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` |
| Adaptive quality | `trackPerf` lowers/raises a `quality` level (0–2) based on rolling frame time |
| Config tables | `DIFFICULTIES`, `THEMES` (stage backdrops), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object (Web Audio, synthesized, no asset files) |
| HUD/feedback | `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit` |
| Input | Keyboard handlers, virtual joystick and FIRE/BOMB buttons, mouse steering |
| Game state | `resetGame`, the `score`/`lives`/`stage`/`kills` globals |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw` |
| Spawning and update | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `update(dt)` |
| Weapons and bombs | `fireWeapon`, `useBomb`, `applyPowerUp` |
| Bosses | `BOSS_PATTERNS` (bullet-pattern generators), `updateBoss`, `killBoss` |
| Lifecycle | `loseLife`, `endGame`, `togglePause`, `loop(now)` (the `requestAnimationFrame` loop) |

Keep changes inside the existing IIFE and follow its conventions. Do not introduce globals.

## Game rules and tuning

These values live in code and are the source of truth. Check the code before quoting them in docs.

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets starting `lives` (4 / 3 / 3), starting `bombs` (3 / 2 / 2), spawn timing, enemy speed and shot chance, and `bossHpMul`.
- **Stages**: each stage needs `KILLS_PER_STAGE` (18) kills, then a boss fight. Stage number drives boss choice (`BOSSES[(stage-1) % 6]`), boss HP (`70 + stage * 35`, scaled by `hpMul` and difficulty), and background theme (`THEMES`).
- **Enemies**: `grunt` (1 HP, from stage 1), `interceptor` (fast, weaving, from stage 1), `tank` (4 HP, from stage 2).
- **Weapons**: two types, `vulcan` and `laser`. Each has 3 levels. A weapon power-up raises the level; past level 3 it switches weapon and resets to level 1.
- **Power-ups**: `life` (6% of drops), `bomb` (8%), `weapon` (20%). Life and bomb are capped by the max values.
- **Scoring**: a kill-chain multiplier starts at x1 and rises by 1 every 5 consecutive kills, capped at x10. The chain breaks after 2.2 seconds without a kill or when the player is hit.
- **Persistence**: the best score is stored in `localStorage`. Mute, difficulty, and reduced-motion preferences are also stored there.

## Conventions

- **Dependencies**: none. Do not add npm packages, CDN scripts, or frameworks unless the user asks for it. Audio is synthesized with Web Audio; do not add sound files.
- **Single file**: keep CSS, markup, and JS in `index.html` unless the user asks for a split.
- **Accessibility and motion**: respect the reduced-motion toggle (`body.reduced-motion`) for any new shake, flash, or pulse effect. Use `announce()` for screen-reader status text.
- **Input parity**: any new control needs a keyboard path and a touch path (`touchControls`). Keep the `touch-action: none` and `user-select: none` behavior intact.
- **Responsive layout**: the canvas scales to the viewport (`BASE_W` 480 × `BASE_H` 720 logical size, set in `resize()`). Keep HUD changes working at phone widths.
- **Commits**: write short, descriptive commit messages in the imperative mood, matching the existing history.

## Git and workflow

- Default working branch for this project's feature work is `claude/gifted-tesla-bvl26l`. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user explicitly asks for one.
