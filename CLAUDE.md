# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web game, **Sky Fighter**: a Raiden-style vertical shoot-em-up played in the browser.

The entire game lives in one file, `index.html` (~1,670 lines): inline CSS in `<style>`, markup in `<body>`, and all game logic in a single inline `<script>` block wrapped in an IIFE. `README.md` is a one-line placeholder.

There is no build step, no package manager, no dependencies, and no automated test suite. To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Repository layout

```
index.html   Entire game: CSS, HTML, JS (canvas 2D rendering, Web Audio SFX)
README.md    Placeholder
CLAUDE.md    This file
```

## Code map (`index.html`)

- **DOM refs and constants** (~lines 426–516): element lookups, `STORAGE_*` localStorage keys, `DIFFICULTIES`, `THEMES` (six stage backdrops), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs), `KILLS_PER_STAGE = 18`.
- **Persistence**: best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under `skyfighter.*` keys. Reads and writes are direct; no try/catch wraps them.
- **Audio** (~543–680): lazily created `AudioContext` (`ensureAudio`), `beep`/`noiseBurst` primitives, and the `sfx` table of named sounds. Audio is created on the first user gesture.
- **HUD and feedback** (~680–750): hearts/bombs rendering, hit/bomb flashes, combo pop, stage banner, screen shake, difficulty switching.
- **Layout scaling** (~753): fixed logical canvas of 480×720 (`BASE_W`/`BASE_H`), scaled to fit the window up to 1.6×.
- **Input** (~768–865): keyboard (`keydown`/`keyup` on `e.code`), touch joystick and FIRE/BOMB buttons, and mouse (steer by moving, click to fire on the canvas).
- **Game state** (~864–912): `resetGame()` initializes player, bullets, enemies, boss, power-ups, score, lives, bombs, chain/multiplier, and stage.
- **Rendering** (~913–1021, ~1507–1640): `drawPlane`, `drawBossShip`, clouds, and the main `draw()`.
- **Simulation** (~1022–1506): `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `update(dt)`, `updateBoss(dt)`, `fireWeapon`, `useBomb`, chain/score helpers, `endGame()`.
- **Main loop** (~1640): `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 50 ms, and calls `update` then `draw` only while `running && !paused`.

## Game rules and controls

- **Difficulty** (`DIFFICULTIES`): easy / normal / hard change spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: each stage ends after `KILLS_PER_STAGE` kills and triggers a boss. The boss cycles through `BOSSES` and the background cycles through `THEMES`. Boss HP scales with stage.
- **Controls**: Arrows/WASD move, Space fires, X or Shift uses a bomb, P or Esc pauses. Mouse, touch joystick, and on-screen buttons are also supported.
- **Accessibility**: a reduced-motion toggle disables screen shake, flashes, and pulses. It defaults from `prefers-reduced-motion` on first visit. The `#srLive` region announces key events to screen readers via `announce()`.
- **Performance**: `trackPerf` samples frame times and sets a `quality` level (0 full, 1 medium, 2 low) to reduce effects when frames run slow.

## Conventions

- Keep the game as a single self-contained file unless the user asks for a restructure. Do not add a bundler, framework, or npm dependencies without a clear reason.
- Preserve the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, and short helper functions.
- Game tuning values belong in the constant tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than inline magic numbers.
- Any new persisted setting should use a `skyfighter.`-prefixed localStorage key declared alongside the existing `STORAGE_*` constants.
- New UI text that changes state should be routed through `announce()` so screen-reader users get the update.
- Keep external resource use to none: the current file loads no external scripts, fonts, or images, and the favicon is an inline data URI. Do not introduce network requests without asking.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a desktop browser and check the start overlay, a full run, pause/resume, game over, and best-score persistence after reload.
2. Check mobile layout with touch emulation: joystick, FIRE, and BOMB.
3. Confirm the browser console shows no errors during play.

If a change touches rendering or input, test it in the browser rather than relying on reading the code alone.

## Git

- Development branch for this work: `claude/gifted-tesla-rkqwz3`.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open pull requests unless explicitly asked.
