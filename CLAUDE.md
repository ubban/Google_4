# CLAUDE.md

Guidance for working in this repository.

## Project overview

This repo is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- `index.html` contains all of the game: markup (~lines 1–420), CSS (`<style>`, ~lines 10–360), and JavaScript (inline `<script>`, ~lines 422–1669).
- `README.md` is a stub and has no setup instructions.

There is no build step, package manager, test suite, linter, or CI config. Open `index.html` in a browser to run the game. Do not add tooling unless asked.

## Architecture

The script runs inside a single IIFE (`(function () { 'use strict'; ... })();`). Nothing is exposed on `window`.

Key sections, in source order:

| Area | What it does |
| --- | --- |
| DOM refs | Caches `document.getElementById` results at the top of the IIFE. |
| Constants | `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`. |
| Audio | Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, `sfx`). No audio files. |
| Adaptive quality | `trackPerf` watches frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low). |
| Layout | `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the window. |
| Input | Keyboard (`keys`, `keydown`/`keyup`), touch joystick, and fire/bomb buttons. |
| World state | `resetGame()` initializes state. `update(dt)` advances the simulation. |
| Bosses | `BOSS_PATTERNS` are bullet-pattern functions. `updateBoss`, `killBoss`, and `spawnBoss` manage them. |
| Rendering | `draw()` renders each frame. `drawPlane`, `drawBossShip`, and `drawClouds` are helpers. |
| Loop | `loop(now)` runs on `requestAnimationFrame`. `dt` is clamped to 0.05s. |

### Controls

- Move: arrow keys or WASD. Mouse: move to steer. Touch: left-side joystick.
- Fire: Space, mouse click, or the on-screen fire button.
- Bomb: `X` or Shift.
- Pause: `P` or Escape.
- Mute and reduced-motion toggles are in the HUD.

### Persistence

State is kept in `localStorage` under the `skyfighter.*` prefix: best score, mute, difficulty, and reduced motion. Reads and writes are direct, with no try/catch. If you change how storage is read, handle the case where `localStorage` is blocked.

### Game rules worth knowing

- Difficulty (`easy` / `normal` / `hard`) sets lives, bombs, spawn rate, enemy speed, enemy fire chance, and boss HP multiplier. Changing one entry in `DIFFICULTIES` changes the balance for that mode only.
- Enemy types unlock by stage through `minStage`. `tank` appears from stage 2.
- Stages advance every `KILLS_PER_STAGE` (18) kills and cycle through `THEMES`. Each stage's boss comes from `BOSSES`.
- Power-up types are `life`, `bomb`, and `weapon`. Weapons toggle between `vulcan` and `laser`.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. The single-file layout is intentional.
- Match the existing style: 2-space indent, single quotes in JS, `const` by default, and short section comments of the form `// ---------- Name ----------`.
- Use `const` for tunable numbers in the config blocks near the top of the script. Don't scatter magic numbers through update or draw code where a config value would be clearer.
- Keep DOM and canvas work separate: game state changes in `update`, pixels in `draw`.
- Preserve accessibility hooks: the `srLive` live region (`announce()`), `prefers-reduced-motion` handling, and ARIA labels on buttons.
- Don't add dependencies, CDN scripts, or external assets. The game is self-contained.

## Making changes

1. Read the relevant section before editing. The file is large, so use `grep -n` for function names.
2. Run the game in a browser and check the affected behavior. There are no automated tests, so manual verification is the only check. Test at both desktop and touch sizes if the change touches input or layout.
3. Check the browser console for errors after each change.
4. Keep diffs focused. Don't reformat unrelated code.

## Git

- Commit and push only when asked.
- Recent commit messages are short and descriptive, e.g. "Add Sky Fighter: Raiden-style vertical shoot-em-up game".
