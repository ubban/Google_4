# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas. The whole game is one self-contained file, `index.html`: markup, CSS, and JavaScript. `README.md` is a placeholder with no project description.

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not invent commands for these. To run the game, open `index.html` in a browser, or serve the directory with any static file server.

## Repository layout

- `index.html` (~1,670 lines), in three parts:
  - Lines ~10–359: a single `<style>` block with all CSS, including the HUD, overlay, touch controls, and the reduced-motion styles.
  - Lines ~362–421: the DOM. `#container` holds the `#gameCanvas`, the HUD (`#hud`, `#topBar`, `#bossBarWrap`), the `#touchControls` layer, and the `#overlay` start, pause, and game-over card.
  - Lines ~422–1667: one `<script>` containing a single IIFE with `'use strict'`. All game state lives inside it.
- `README.md`: a stub.

## Architecture (inside the script IIFE)

The file is organized into commented sections, roughly in this order:

1. **DOM references and constants.** Element lookups are `const`s at the top. `STORAGE_*` keys are defined here.
2. **Adaptive performance** (`trackPerf`, `quality`). Measures the average frame time over 45 frames and steps `quality` between 0 (full), 1 (medium), and 2 (low).
3. **Tuning tables.** `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, and boss HP multiplier for easy, normal, and hard), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes, cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses, cycled by stage).
4. **Persisted settings.** Best score, mute, difficulty, and reduced motion are read from `localStorage` at startup.
5. **Audio.** Web Audio, created lazily by `ensureAudio()`. The `sfx` object holds named sounds. Audio starts only from a user gesture.
6. **HUD helpers.** `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, and `announce` (for screen readers, via `#srLive`).
7. **Input.** Keyboard (`keys[e.code]`), touch joystick and buttons, and mouse steering and firing on the canvas.
8. **Game state.** Top-level `let`s for score, lives, bombs, stage, chain and multiplier, and the player, enemy, bullet, and power-up arrays. `resetGame()` reinitializes them.
9. **Spawning and weapons.** `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, and `fireWeapon`. Weapon patterns are in `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`. Boss attacks are the `BOSS_PATTERNS` array.
10. **Update and draw.** `update(dt)` advances the simulation. `draw()` renders the frame. `loop(now)` drives both through `requestAnimationFrame`.
11. **Lifecycle.** `loseLife`, `endGame`, `killBoss`, `togglePause`, and the `startBtn` handler.

Game loop: `loop` clamps `dt` to 0.05 s. It calls `update` and then `draw` only when `running && !paused`.

## Controls

- Move: arrow keys or WASD, or the mouse on the canvas.
- Fire: Space, hold the mouse button, or hold the on-screen FIRE button.
- Bomb: X or Shift, or the BOMB button.
- Pause: P or Esc, or the pause button.
- Touch: drag in the left zone for a virtual joystick.

Space's default is prevented so the page does not scroll.

## Gameplay rules

- Lives and bombs depend on difficulty. Losing a life grants 1.5 s of invulnerability and breaks the kill chain.
- Power-ups are `life`, `bomb`, and `weapon`. Weapon power-ups raise the level from 1 to 3, then switch between `vulcan` and `laser` and reset to level 1.
- Bombs are capped by `maxBombs`. Lives are capped by `maxLives`.
- Each stage ends after `KILLS_PER_STAGE` kills, followed by a boss. Defeating a boss adds 500 points and advances the stage.
- Score is multiplied by the chain multiplier. It rises by 1 for every 5 chained kills and caps at 10×.

## Persisted state

All keys use the `skyfighter.` prefix:

| Key | Values |
| --- | --- |
| `skyfighter.bestScore` | number |
| `skyfighter.muted` | `'1'` or absent |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'`, `'0'`, or absent (falls back to `prefers-reduced-motion`) |

Every `localStorage` access can throw in private browsing or when storage is blocked. If you add new reads or writes, wrap them in `try/catch` and keep the game working without them. The existing calls do not do this, so treat them as a known gap rather than a pattern to copy.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not add a build step without discussing it first.
- Stay inside the existing IIFE. Do not add new globals.
- Use `const` by default and `let` for state that changes. Keep `'use strict'`.
- Match the existing style: 2-space indentation, single quotes in JS, and `// ---------- Section ----------` dividers for major sections.
- Put new tuning values in the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Keep accessibility hooks working: `announce()` for significant events, the `reduced-motion` body class, and `#srLive`.
- Keep the layout usable at phone width. Touch controls are part of the game, not an afterthought.

## Testing and verification

There is no automated test suite. Verify changes by loading `index.html` in a browser and playing:

- Start, pause, resume, and game over.
- Each difficulty.
- Bomb use, power-up pickup, and boss fights.
- Mute and reduced-motion toggles, and their persistence across reloads.
- Keyboard, mouse, and touch controls.

Check the browser console for errors. Note in your summary which of these you actually exercised.

## Git

- Development branch: `claude/gifted-tesla-sat6fk`. Commit and push there.
- Do not open a pull request unless someone explicitly asks for one.
