# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a single-file, browser-based vertical shoot-em-up (Raiden-style) written in plain HTML, CSS, and JavaScript. There is no build step, no package manager, no framework, and no test suite.

### Repository layout

| Path        | Purpose                                                         |
|-------------|-----------------------------------------------------------------|
| `index.html`| The entire game: markup (~360 lines), inline `<style>`, and one inline `<script>` IIFE (~1,250 lines) |
| `README.md` | Minimal placeholder                                             |

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Nothing needs to be installed.

## Verifying changes

- There are no automated tests, linters, or build scripts configured.
- Verify changes by playing the game in a browser. At minimum, exercise: start, pause/resume (P / Esc), a bomb (X / Shift), each difficulty, a boss fight, game over, and the high-score persistence after a reload.
- Check touch controls and the mobile layout when touching input or layout code. The canvas resizes via `resize()`.
- Check the browser console for errors after any change.

## Architecture of `index.html`

### Markup and styling

- The HTML sets up the `<canvas id="gameCanvas">`, an overlay (`#overlay`) for the title / game-over screens, a HUD (score, best, multiplier, weapon, hearts, bombs), a boss health bar, touch controls (`#touchControls`, joystick, fire and bomb buttons), and an `aria-live` region (`#srLive`) for screen-reader announcements.
- Colors and styling live in the inline `<style>` block. Respect `body.reduced-motion`, which disables or tones down motion effects.

### Script structure

All logic is inside one `(function () { 'use strict'; ... })();` IIFE. Nothing is exposed on `window`. Keep new code inside this scope unless there is a clear reason not to.

The script is organized into sections marked with comments of the form `// ---------- Section name ----------`. Follow that convention when adding a section. The main sections, in order, are:

1. **DOM references and storage keys**: element lookups are `const`s at the top.
2. **Adaptive performance** (`trackPerf`): averages recent frame times and steps a `quality` level (0 full, 1 medium, 2 low) up or down.
3. **Tunables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`.
4. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` mapping. Audio is created lazily on the first user gesture (`ensureAudio()`), which browsers require.
5. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
6. **Difficulty and layout**: `setDifficulty`, `resize` (uses `BASE_W = 480`, `BASE_H = 720` logical coordinates).
7. **Input**: keyboard (`keys` map plus handlers), joystick for touch, mouse steering via `canvas` events, fire and bomb buttons.
8. **Game state**: `resetGame`, drawing helpers (`drawPlane`, `drawBossShip`), spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), collision (`rectsOverlap`), particles (`spawnExplosion`), scoring (`addScore`, `breakChain`), weapons (`fireWeapon`, `applyPowerUp`), and bombs (`useBomb`).
9. **Update and boss AI**: `update(dt)` is the per-frame simulation. Boss attacks are defined as functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) and selected through the `BOSS_PATTERNS` array, driven by `updateBoss`.
10. **Lifecycle**: `loseLife`, `killBoss`, `endGame`, `togglePause`.
11. **Rendering**: `draw()` and `drawClouds`.
12. **Main loop**: `loop(now)` clamps `dt` to 0.05 s, runs `update` and `draw` only when `running && !paused`, then calls `requestAnimationFrame(loop)`.

### Game loop and timing

- Simulation is time-based: `update(dt)` receives seconds, and `dt` is clamped to 0.05 s to avoid large jumps after tab switches. Keep new movement and timers in seconds multiplied by `dt`.
- Drawing happens every frame after `update`, and only while the game is running and not paused.

### Persistence

`localStorage` keys (all prefixed `skyfighter.`):

| Key                        | Value                              |
|----------------------------|------------------------------------|
| `skyfighter.bestScore`     | Numeric high score                 |
| `skyfighter.muted`         | `'1'` when muted                   |
| `skyfighter.difficulty`    | `'easy'`, `'normal'`, or `'hard'`  |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; defaults to the `prefers-reduced-motion` media query when unset |

Reads and writes are not wrapped in `try/catch`. If you add new storage, use the same `skyfighter.` prefix and consider failures in private browsing or with blocked storage.

## Conventions

- **Single file, no dependencies.** Do not add libraries, a bundler, or a separate build step unless the user asks for it.
- **Match the existing style.** Two-space indentation, single quotes for strings, `const` by default, arrow functions for callbacks, and section-divider comments as above. Keep comments brief and explain *why*, not *what*.
- **Tunables live in the constant tables.** Enemy stats, boss HP multipliers, difficulty values, and stage palettes belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES`, not hard-coded inside update logic.
- **Accessibility.** Announce important state changes through `announce()` (the `#srLive` region), keep the `reduced-motion` class and setting working, and keep the keyboard and touch controls functional.
- **Audio** must only start from a user gesture. Go through `ensureAudio()` rather than creating an `AudioContext` directly.
- **Coordinates** are in the logical `BASE_W` × `BASE_H` space. Convert pointer input through `canvasPointFromEvent` rather than using raw client coordinates.

## Git

- Development branch for this repository is `claude/gifted-tesla-nm9tot`.
- Recent history: `Initial commit`, then `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.
- Write clear, descriptive commit messages in the imperative mood.
