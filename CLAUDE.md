# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-page, Raiden-style vertical shoot-'em-up
built on an HTML5 `<canvas>`. There is no build step, no package manager, and no
test suite. The entire game lives in one file:

- `index.html` (~1,670 lines): inline `<style>`, markup, and one inline `<script>`.
- `README.md`: a one-line stub.

Open `index.html` directly in a browser to play. Nothing needs to be installed.

## Repository layout

```
index.html   # The whole game: CSS, HTML, and JS in one file
README.md    # Minimal project stub
CLAUDE.md    # This file
```

Keep the game as a single self-contained file unless the user asks for a split.
Do not introduce a bundler, framework, or external CDN dependency without asking.

## Architecture of `index.html`

The inline script is a single IIFE-style block with these major sections, roughly in
file order:

1. **DOM references**: `const xxxEl = document.getElementById(...)` at the top of the script.
2. **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`
   (all prefixed `skyfighter.`). Reads/writes go through `localStorage` directly.
3. **Tuning data (constants)**:
   - `DIFFICULTIES` (easy / normal / hard): spawn rate, enemy speed, enemy fire chance,
     starting lives, starting bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed before a boss spawns.
   - `THEMES`: six sky palettes, cycled by `(stage - 1) % THEMES.length`.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). Each has a `minStage`.
   - `BOSSES`: six bosses, each with colors and an `hpMul`.
4. **Audio**: lazily created `AudioContext` (`ensureAudio`). All sound is synthesized
   with oscillators and noise bursts (`beep`, `noiseBurst`); there are no audio files.
   `sfx` is the table of named sound effects. Mute state is persisted.
5. **Rendering helpers**: `drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion`.
   Drawing happens on the single `ctx` from the `gameCanvas` element.
6. **Input**: keyboard (`keys` map, `keydown`/`keyup`), mouse (steer and fire on the
   canvas), and touch (virtual joystick in `moveZone`, `fireBtn`, `bombBtn`). Touch is
   detected via `isTouchDevice`.
7. **Game state**: module-level `let`s (`player`, `bullets`, `enemies`, `enemyBullets`,
   `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`, `multiplier`, ...).
   `resetGame()` reinitializes all of it.
8. **Simulation**: `update(dt)` advances entities, handles collisions and stage progress.
   `updateBoss(dt)` drives boss phases. Boss attack patterns are the `fire*` functions
   (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`,
   `fireHomingSeeded`, `fireDoubleRing`).
9. **Weapons and bombs**: `fireWeapon()` implements the Vulcan spread (level 1 to 3) and
   the Laser (`player.weapon` is `'vulcan'` or `'laser'`; a power-up toggles between them).
   `useBomb()` clears the screen and damages bosses. Bombs start at `maxBombs` on a new game
   (`resetGame`), and the bomb power-up adds one, capped at `maxBombs`.
10. **Flow**: `endGame()` (saves best score), `togglePause()`, `showStageBanner`.
11. **Main loop**: `draw()` renders each frame, then `loop(now)` runs via
    `requestAnimationFrame` with a delta-time `dt`.

### Key gameplay rules (so changes stay consistent)

- Stages advance after `KILLS_PER_STAGE` kills; a boss is spawned ~1.2 s later.
  Defeating a boss increments `stage` and resets `bossWarned`.
- Tank and later enemy types are gated by `minStage`.
- Score uses a kill-chain multiplier (`chain`, `chainTimer`, `multiplier`).
- Power-ups include weapon level-ups, the Vulcan/Laser toggle, bombs, and extra life (capped at `maxLives`).
- A performance monitor (`trackPerf`, `quality` 0 to 2) lowers rendering cost on slow devices.
  Keep new effects gated behind `quality` where they are expensive.
- Reduced motion (`reducedMotion`) suppresses screen shake and flashes. Respect it for any new
  visual effect that moves the screen or flashes.

## Controls (keep the in-game hint in sync)

The hint is in `#controlsHint` (around `index.html:417`). If you change a control, update it.

| Action        | Keyboard                  | Mouse / touch                    |
|---------------|---------------------------|----------------------------------|
| Move          | Arrows / WASD             | Mouse move; drag left (touch)    |
| Fire          | Space (hold)              | Click / hold; tap FIRE           |
| Bomb          | X or Shift                | Tap BOMB                         |
| Pause         | P or Esc                  | Pause button                     |

## Persistence

| Key                       | Value                              |
|---------------------------|------------------------------------|
| `skyfighter.bestScore`    | Highest score (number)             |
| `skyfighter.muted`        | `'1'` when muted                   |
| `skyfighter.difficulty`   | `easy` \| `normal` \| `hard`       |
| `skyfighter.reducedMotion`| `'1'` or `'0'` (absent = use OS)   |

Wrap any new storage access so the game still works if `localStorage` is blocked
(private mode, disabled storage).

## Conventions

- **Plain JavaScript and CSS only.** ES2017-level syntax is fine; match the surrounding style
  (2-space indent, single quotes in JS, `const` by default).
- **Keep the single-file structure.** Add new constants next to the existing tuning tables;
  add new functions near the section they belong to.
- **Mobile first.** The layout scales the canvas to fit the viewport (`resize()`, base
  resolution 480x720, max scale 1.6). Verify changes at phone width and in landscape.
- **Accessibility.** Announcements go through `announce()` (the `srLive` region). Buttons
  need text or `aria-label`s. Respect `prefers-reduced-motion`.
- **No external assets.** Graphics are drawn on canvas and audio is synthesized. Do not add
  image, audio, or font files unless asked.

## Testing and verification

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser (or serve the directory with any static server).
2. Play through at least one stage, a boss fight, a bomb, a pause/resume, and a game over.
3. Check the high score persists across reloads and that mute and difficulty settings stick.
4. For layout changes, check a narrow phone viewport and a landscape viewport.

A Playwright run against `file://.../index.html` is a reasonable way to smoke-test: it can
start the game, press keys, and capture a screenshot.

If you add a test harness, document how to run it here.

## Git workflow

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-mh3jv0`).
- Use clear, descriptive commit messages that explain *why* the change is needed.
- Do not open a pull request unless the user asks for one.
- Do not push to other branches without explicit permission.

## Working notes for Claude

- Read the relevant section of `index.html` before editing; the file is large, so use
  targeted `Grep`/`Read` ranges rather than loading everything.
- Prefer small, surgical edits over restructuring the single-file layout.
- When changing gameplay numbers, change them in the tuning tables (`DIFFICULTIES`,
  `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`) rather than hard-coding them in logic.
- Do not commit generated or temporary files. Use the scratchpad directory for scratch work.
