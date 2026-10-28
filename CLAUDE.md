# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` holds **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- `index.html` is the entire game: markup, CSS, and one inline `<script>` (an IIFE with `'use strict'`).
- `README.md` is a stub with no project description.
- There is no build step, package manager, bundler, test suite, or linter. Nothing needs installing.

To run the game, open `index.html` in a browser. Serving it with any static server also works.

## Repository layout

```
index.html   # The whole game (HTML + <style> + <script>)
README.md    # Stub
CLAUDE.md    # This file
```

Inside the `<script>` block, the code is grouped by section comments. Look for these when navigating:

| Area | Key identifiers |
| --- | --- |
| Persistence keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` |
| Adaptive quality | `quality`, `trackPerf` (drops quality when frame time is high) |
| Tuning tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` |
| Input and layout | `keys`, joystick helpers, `canvasPointFromEvent`, `resize` (base resolution `BASE_W` x `BASE_H` = 480 x 720) |
| Gameplay | `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update` |
| Bosses | `BOSS_PATTERNS`, `updateBoss`, `killBoss` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw` |
| Main loop | `loop` |

## Game mechanics

- **Difficulty** (`easy`, `normal`, `hard`) sets lives, bombs, spawn rate, enemy shoot chance, and boss HP multiplier.
- **Stages**: every `KILLS_PER_STAGE` (18) kills triggers a boss. Each stage uses the next entry in `BOSSES` and the next entry in `THEMES`, both cycling by `(stage - 1) % length`. Boss HP is `70 + stage * 35`, scaled by the boss's `hpMul` and the difficulty's `bossHpMul`.
- **Enemies**: `grunt`, `interceptor` (weaves), and `tank` (4 HP, unlocked at stage 2).
- **Weapons**: `vulcan` and `laser`, each with 3 levels. A weapon power-up raises the level. At level 3 it cycles to the other weapon and resets to level 1.
- **Power-ups**: `life`, `bomb`, and `weapon`, dropped at random from destroyed enemies.
- **Score**: kill chains build a multiplier (`breakChain` resets it).
- **Controls**: mouse, arrows/WASD, or touch joystick to move; Space or click to fire; X or Shift to bomb; P or Esc to pause.

## Conventions

- **Keep it a single file.** New code goes in `index.html` unless the user asks for a split. Do not add a build step or dependencies without asking.
- **Match the existing style.** The code uses 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for callbacks, and short section comments such as `// ---------- Section ----------`.
- **Tune in tables.** Balance changes belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES`, not inline magic numbers.
- **Preserve accessibility hooks.** Game-state changes are announced through `announce()` into the `srLive` live region. Respect the `reduced-motion` class and `reducedMotion` flag when adding effects such as shake or flashes.
- **Keep it mobile-safe.** Layout must work at phone width. Keep `touch-action` and the viewport meta settings intact.
- **Storage access.** `localStorage` reads and writes are unguarded. If you add new keys, follow the `skyfighter.*` prefix convention.

## Verifying changes

There are no automated tests. After a change:

1. Open `index.html` in a browser, or run it with the pre-installed Playwright Chromium if you need a headless check.
2. Start a game, then check movement, firing, a bomb, pausing, and game over.
3. Reach a boss if the change touches stages, bosses, or weapons. Temporarily lowering `KILLS_PER_STAGE` is a quick way to get there.
4. Check the HUD and overlay at a narrow (phone-width) viewport.

## Git

- Develop on the branch the session names. Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood.
