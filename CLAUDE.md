# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The whole game lives in one file:

- `index.html` (~1670 lines): inline `<style>`, the HTML markup, and one inline `<script>` wrapped in an IIFE with `'use strict'`.
- `README.md`: a stub with only the project name. Keep it minimal unless the user asks for more.

There is no `package.json`, bundler, framework, test suite, linter, or CI configuration. Do not invent build or test commands. To run the game, open `index.html` in a browser, or serve the directory with any static file server.

## Repository layout

```
index.html   # The entire game: markup, CSS, and JavaScript
README.md    # Project title stub
```

## Architecture of `index.html`

The script is organized in commented sections. Keep new code in the matching section.

| Area | What it covers |
| --- | --- |
| DOM lookups | `const xxxEl = document.getElementById(...)` at the top of the IIFE |
| `STORAGE_*` keys | `localStorage` keys, all prefixed `skyfighter.` |
| `// ---------- Adaptive performance ----------` | `trackPerf` measures frame time over 45 frames and steps a `quality` level (0 full, 1 medium, 2 low) |
| `DIFFICULTIES` | Per-difficulty tuning: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier |
| `KILLS_PER_STAGE`, `THEMES` | Stage length (18 kills) and the sky/cloud/sun palettes cycled per stage |
| `ENEMY_TYPES`, `BOSSES` | Enemy stats (`grunt`, `interceptor`, `tank`) and boss definitions (`VANGUARD` through `TITAN`) |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is created lazily on the first user gesture. |
| Drawing helpers | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp` |
| Weapons | `fireWeapon` (Vulcan spread or Laser lanes, levels 1 to 3), `useBomb` |
| Boss patterns | `BOSS_PATTERNS` and the `fire*` helpers (`fireRadial`, `fireAimedSpread`, `fireCross`, and others) |
| Game state | `resetGame`, `update(dt)`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause` |
| Input | Keyboard `keydown`/`keyup`, mouse and pointer handling, and the touch joystick |
| Loop | `loop(now)` clamps `dt` to 0.05 s, then calls `update` and `draw` while running and not paused |

### Core concepts

- **Game loop**: `requestAnimationFrame(loop)` drives everything. Update and draw are skipped while paused or not running. Time is in seconds and is clamped, so logic should use `dt` rather than frame counts.
- **Scoring**: `addScore` applies a kill-chain multiplier. `breakChain` resets it.
- **Power-ups** (`applyPowerUp`): `life`, `bomb`, and `weapon`. Weapon power-ups raise the level from 1 to 3. After level 3 they toggle between `vulcan` and `laser`.
- **Stages**: enemy waves run until `KILLS_PER_STAGE` is reached, then a boss spawns. Each stage picks a `THEMES` entry.
- **Persistence**: best score, mute, difficulty, and reduced-motion preference use `localStorage`. Each read and write is a plain call without a try/catch, so keep that in mind if you add more storage.
- **Accessibility**: `announce()` writes to the `#srLive` live region. Reduced motion (`body.reduced-motion`) disables screen shake, flash, and pulse effects. It defaults to the `prefers-reduced-motion` media query and can be toggled.
- **Responsive and touch**: `resize()` fits a `BASE_W` x `BASE_H` (480 x 720) logical canvas to the viewport. Touch controls appear only on touch devices.

## Conventions

- **Style**: match the existing code. Use 2-space indentation, single quotes in JavaScript, arrow functions for callbacks, `const` by default, and `let` only for reassigned state.
- **Naming**: functions are camelCase verbs (`spawnEnemy`, `applyPowerUp`). Constants are UPPER_SNAKE_CASE (`STORAGE_BEST`, `BASE_W`). DOM handles end in `El` or are named after the element (`startBtn`).
- **Section headers**: use the `// ---------- Name ----------` style to group related code.
- **Comments**: keep them short and explain why, not what. The file is mostly self-explanatory.
- **Single file**: keep the game in `index.html`. Do not split it into modules or add a build step unless the user asks for that.
- **No dependencies**: the project uses no external libraries. Do not add CDN scripts or packages without asking.
- **Mobile**: the layout must still work at phone width. Test changes to HUD, overlay, or touch code with a narrow viewport.

## Making changes

1. Read the relevant section of `index.html` before editing. The file is large, so search for the function or constant name first.
2. Keep changes minimal and scoped to the request.
3. Verify in a browser. Since there are no automated tests, load the page, start a game, and exercise the change. Check the browser console for errors. Check both desktop keyboard and touch layouts when input or layout code changes.
4. When tuning gameplay, change values in `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` rather than hard-coding numbers in logic.

## Git

- Development branch for this work: `claude/gifted-tesla-km2nlb`. Default branch: `main`.
- Write clear, imperative commit messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.
