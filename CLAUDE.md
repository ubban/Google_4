# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game. The whole game lives in `index.html`: inline CSS in `<style>`, markup in `<body>`, and one inline `<script>` IIFE.

There is no build step, no package manager, no bundler, and no third-party dependencies. Do not add any of these unless the user asks.

## Repository layout

```
index.html   Entire game: CSS (lines ~10-359), markup (~361-421), JS (~422-1667)
README.md    One-line project title
CLAUDE.md    This file
```

## Running and verifying

- Run the game by opening `index.html` in a browser. Serving it over a static server also works.
- No test suite, linter, or formatter is configured. Verify changes by playing the game in a browser, and say so in your summary if you could not do that.
- Check both desktop (mouse and keyboard) and touch layouts when you change input, HUD, or overlay code.

## Code structure inside `index.html`

The script is wrapped in `(function () { 'use strict'; ... })();`. Keep new code inside that wrapper.

Sections are marked with comment banners such as `// ---------- Input ----------`. Follow that style when you add a section.

Key areas, roughly in file order:

- **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, and `STORAGE_REDUCED`, all prefixed `skyfighter.`. Read and write them through `localStorage`.
- **Adaptive quality**: `trackPerf` samples frame times and steps `quality` (0 full, 1 medium, 2 low) to keep the frame rate up.
- **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (one sky palette per stage), `ENEMY_TYPES`, and `BOSSES`. Balance changes should usually go here rather than in the logic.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is created lazily on the first user gesture, which browsers require.
- **Canvas sizing**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the window, capped at 1.6x. Draw in base coordinates.
- **Input**: keyboard (`keydown` and `keyup` store state in `keys[e.code]`), touch (`joystickStart`, `joystickMove`, `joystickEnd`, fire and bomb buttons), and mouse (`canvasPointFromEvent`, which steers and fires).
- **Game state and loop**: `resetGame`, `update(dt)`, `draw()`, and `loop(now)`. `dt` is clamped to 0.05 s. `update` runs only while `running && !paused`.
- **Weapons and power-ups**: `fireWeapon`, `applyPowerUp`, `maybeSpawnPowerUp`, and `weaponLabel`.
- **Bosses**: `spawnBoss`, `updateBoss`, `killBoss`, and `BOSS_PATTERNS`, which holds one attack-pattern list per boss design. Patterns are built from helpers such as `fireRadial`, `fireAimedSpread`, `fireCross`, and `fireHomingSeeded`.
- **Lives, bombs, and game over**: `loseLife`, `useBomb`, `endGame`, and `togglePause`.

## Controls (keep the in-game hint in sync)

The hint text is in `#controlsHint`, around line 417.

- Desktop: mouse to steer, click to fire. Arrow keys or WASD to move. Space to fire. X or Shift for a bomb. P or Esc to pause.
- Touch: drag left to fly, tap FIRE, tap BOMB.

If you change a binding, update the hint text and the `keydown` handler together.

## Conventions

- Use plain JavaScript (ES2015+) and `const` and `let`. Match the existing style: two-space indentation, single quotes, arrow functions for callbacks.
- Keep the code in one file unless the user asks for a split.
- Use the existing `announce()` helper to update the `aria-live` region for significant game events. Keep the accessibility features (reduced motion, screen-reader announcements) working when you add effects.
- Respect the reduced-motion setting. Screen shake, flashes, and heavy particle effects should check `reducedMotion`.
- Comments explain *why*, not *what*. Keep them short.

## Git

- Develop on the branch the session names (currently `claude/gifted-tesla-8mp303`). Do not push to `main` without explicit permission.
- Write commit messages that describe the gameplay or code change, with a short body for anything non-obvious.
- Do not open a pull request unless the user asks.
