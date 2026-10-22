# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project containing a single browser game, **Sky Fighter** (a Raiden-style vertical shoot-'em-up). There is no build system, package manager, framework, or test suite.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: inline CSS, HTML markup, and one IIFE `<script>` (~1,670 lines) |
| `README.md` | Placeholder; contains only the project name and a timestamp     |

Git: the default branch is `main`. Feature work for this session goes on `claude/gifted-tesla-9efw4l`.

## Running it

No build step. Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There are no lint, test, or type-check commands configured. Verify changes by loading the page and playing it.

## Code layout inside `index.html`

- **`<style>`** (around lines 10–360): all styling. Layout is a fixed-aspect canvas centered in `#container`, with HUD, boss bar, touch controls, and the overlay card layered on top.
- **`<body>`** (around lines 361–421): DOM skeleton. Key IDs: `gameCanvas`, `hud`, `topBar` (mute / reduced-motion / pause buttons), `touchControls` (`moveZone`, `fireBtn`, `bombBtn`), `overlay` (title card, difficulty buttons, `startBtn`), `srLive` (screen-reader announcements).
- **`<script>`** (around lines 422–1668): a single `(function () { 'use strict'; ... })();` wrapper. Nothing is exported to `window`. Sections, in order:
  1. DOM lookups and `STORAGE_*` localStorage keys (`skyfighter.*`).
  2. Adaptive performance (`trackPerf`) that steps a `quality` level (0–2) based on average frame time.
  3. Tuning tables: `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE` (18).
  4. Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Web Audio is created lazily on the first user gesture.
  5. HUD and effects helpers: hearts/bombs rendering, flashes, combo pop, stage banner, screen shake.
  6. Input: keyboard (`keys` map), touch joystick and buttons, and mouse steering on the canvas.
  7. Game state: `resetGame`, `update(dt)`, `draw()`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), collisions, scoring with a combo multiplier (`addScore`, `breakChain`), and bombs (`useBomb`).
  8. Weapons and power-ups: `fireWeapon` (`vulcan` / `laser`), `applyPowerUp` (`life`, `bomb`, `weapon`).
  9. Bosses: `BOSS_PATTERNS` is an array per boss of attack functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, and so on), driven by `updateBoss` and `killBoss`.
  10. Game flow: `loseLife`, `endGame` (saves best score), `togglePause`.
  11. Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`.
  12. Main loop: `loop(now)` calls `requestAnimationFrame`. `dt` is clamped to 50 ms. The loop runs `update` and `draw` only while the game is running and not paused.

Rendering is done on a 480×720 logical canvas (`BASE_W` / `BASE_H`). `resize()` scales the CSS size to fit the window, capped at 1.6×.

## Conventions to follow

- **Single file, no dependencies.** Keep all game code in `index.html` unless the user asks for a split. Don't add a framework, bundler, or CDN script without a clear reason.
- **Match the existing style.** Two-space indentation, single quotes, `const` / `let`, arrow functions for callbacks, and short `// ---------- Section ----------` banner comments between sections. Keep comments sparse and explanatory.
- **Keep it in the IIFE.** New globals should not be added to `window`. Helpers belong inside the wrapper.
- **Tune through the tables.** Difficulty, enemy stats, boss HP multipliers, and stage themes live in the constant objects near the top of the script. Prefer editing those over hard-coding numbers in logic.
- **Persist only through the existing keys.** Use the `skyfighter.*` localStorage keys, and wrap any new storage access in a way that tolerates it being unavailable (private mode, blocked storage).
- **Respect reduced motion and mute.** Screen shake, flashes, and pulses go through the existing helpers, which check `reducedMotion`. Sound goes through `sfx` / `beep`, which check `muted`.
- **Support touch and keyboard.** Any new control should work with both the keyboard/mouse bindings and the on-screen touch controls, and the controls hint in `#controlsHint` should stay accurate.
- **Accessibility.** Announce important state changes through `announce()` into `#srLive`.

## Controls (for reference when editing input code)

- Move: mouse (on canvas), Arrow keys or WASD, or the touch joystick.
- Fire: click / hold on canvas, Space, or the FIRE button.
- Bomb: X or Shift, or the BOMB button.
- Pause: P or Escape, or the pause button.

## Persistence keys

| Key                          | Meaning                              |
|------------------------------|--------------------------------------|
| `skyfighter.bestScore`       | Highest score reached                |
| `skyfighter.muted`          | `'1'` when sound is muted            |
| `skyfighter.difficulty`     | `easy` / `normal` / `hard`           |
| `skyfighter.reducedMotion`  | `'1'` / `'0'`; defaults to the OS `prefers-reduced-motion` setting |

## Working on this repo

- Read the relevant section of `index.html` before editing; the file is large, so search for function names rather than reading it top to bottom.
- Test changes by opening the page in a browser and playing through at least one stage, including a boss fight, when touching combat, bosses, or power-ups.
- If you add files, update the file table above.
