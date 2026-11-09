# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html` (inline `<style>` and inline `<script>`, drawn on a single `<canvas id="gameCanvas">`). `README.md` is a placeholder and has no setup instructions.

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Do not add any of these unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static file server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## File layout

- `index.html` — markup, CSS, and game logic. Sections, in order:
  - `<style>`: layout, HUD, overlays, touch controls, `body.reduced-motion` overrides.
  - `<body>`: canvas, HUD elements (score, best, multiplier, weapon, hearts, bombs, boss bar), overlay with difficulty buttons, touch controls, pause/mute/motion buttons, an `aria-live` region (`#srLive`).
  - `<script>` inside an IIFE with `'use strict'`. Game state and logic are module-scoped; nothing is exported to `window`.
- `README.md` — placeholder.
- `CLAUDE.md` — this file.

## Architecture of the game script

Constants and tables are defined near the top of the script:

- `DIFFICULTIES` — `easy`, `normal`, `hard`. Each sets spawn timing (`spawnBase`, `spawnMin`), `enemySpeed`, `enemyShootChance`, starting `lives`, starting `bombs`, and `bossHpMul`.
- `KILLS_PER_STAGE` — kills needed before a boss spawns (18).
- `THEMES` — six background palettes, cycled by `(stage - 1) % THEMES.length`.
- `ENEMY_TYPES` — `grunt`, `interceptor` (weaves), `tank` (4 HP, unlocked at stage 2). `minStage` gates spawns.
- `BOSSES` — six boss designs, cycled by `(stage - 1) % BOSSES.length`. HP scales with stage (`70 + stage * 35`) and with the boss's and difficulty's multipliers.

Main systems, all as functions inside the IIFE:

- **Loop**: `loop(now)` drives `update(dt)` and `draw()`. `togglePause()` pauses it.
- **Input**: `keys` (keyed by `KeyboardEvent.code`), touch joystick (`joystickStart/Move/End`), fire and bomb buttons. Desktop keys: arrows or WASD to move, Space to fire, X or Shift to use a bomb, P or Escape to pause.
- **Weapons**: `player.weapon` is `'vulcan'` or `'laser'`, toggled by weapon power-ups. `fireWeapon()` handles firing cadence, and `useBomb()` clears the screen.
- **Boss patterns**: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`. `updateBoss()` picks among them. `killBoss()` advances the stage.
- **Power-ups**: `maybeSpawnPowerUp()` and `applyPowerUp()` handle `life`, `bomb`, and `weapon` types.
- **Audio**: a Web Audio synth (`beep`, `noiseBurst`, and the `sfx` table). `ensureAudio()` must run from a user gesture before audio starts.
- **Adaptive quality**: `trackPerf()` samples frame times over 45 frames and steps `quality` (0 full, 1 medium, 2 low) to keep frame rate up. Rendering checks `quality`.
- **Accessibility**: `announce()` writes to the `aria-live` region. Reduced motion is a user setting that adds the `reduced-motion` class to `<body>`.

### Persistence

Settings and scores are stored in `localStorage` under these keys. Keep the names stable, because they are how players' saved state is found:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split. If you do split, update this file's layout section.
- Use plain JavaScript (ES2017+ features already in use: `const`/`let`, arrow functions, template literals, `classList`). Do not introduce a framework or a build step.
- Keep the IIFE and `'use strict'` wrapper. Do not leak globals.
- Match the existing style: two-space indentation, single quotes in JS, section comments like `// ---------- Input ----------`.
- Time-based logic uses the `dt` passed to `update()`. Do not tie movement to frame count.
- Any new HUD text or state change that matters to players should also go through `announce()` so screen readers get it.
- Respect `reduced-motion`: new screen shake, flashes, or animations need a matching check.
- Touch support is first-class. Any new control needs a touch path as well as a keyboard path, and must work with `touch-action: none`.

## Verifying changes

There are no automated tests. To verify a change:

1. Open `index.html` in a browser (or serve it locally).
2. Play through the easy, normal, and hard difficulties at least briefly.
3. Check that stage progression works: 18 kills spawn a boss, the boss defeat advances the stage, and the theme and boss change.
4. Check the touch controls with browser device emulation, and check the layout at phone width.
5. Check the `reduced-motion` toggle and the mute toggle.
6. Confirm the best score persists across reloads.

If you add logic that is easy to get wrong, consider writing a small standalone check, but do not add a test framework without asking.

## Git workflow

- Develop on the branch the session specifies. Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless asked.
