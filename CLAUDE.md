# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a
single self-contained HTML5 canvas page.

- `index.html` is the entire game: markup (~lines 1–421), CSS (inline `<style>`),
  and one inline `<script>` (~lines 422–1667). There is no build step, no
  bundler, no `package.json`, and no test suite.
- `README.md` is a placeholder; it has no setup instructions.
- Git: default branch is `main`. Feature work for this session goes on
  `claude/gifted-tesla-qdpoe2`.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static
  server (e.g. `python3 -m http.server`). No install step is needed.
- There are no automated tests or linters configured. The fastest check after
  a JS edit is to extract the script and parse it:
  ```bash
  awk 'NR>=423 && NR<=1666' index.html > /tmp/game.js && node --check /tmp/game.js
  ```
  (Re-check the line range first if the file has changed, because the script
  block's start and end lines move with edits.)
- For behaviour changes, play-test in a real browser. Check both desktop
  (keyboard/mouse) and touch layouts, since the HUD and touch controls are
  responsive.

## Code layout inside `index.html`

The script is one IIFE-style block with these main parts, in order:

1. **DOM refs and constants**: `STORAGE_*` keys, `DIFFICULTIES`,
   `KILLS_PER_STAGE` (18), `THEMES` (sky palettes per stage, cycled),
   `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs, cycled).
2. **Persistence and settings**: best score, mute, difficulty, and reduced
   motion are read from and written to `localStorage` (see below).
3. **Audio**: a lazily created `AudioContext` (`ensureAudio`) plus
   synthesized SFX built from `beep` and `noiseBurst`. Nothing is loaded from
   files.
4. **Visual feedback**: HUD rendering (`renderHearts`, `renderBombs`), screen
   shake, hit/bomb flashes, combo pops, stage banners.
5. **Responsive canvas**: `resize()` scales a fixed `480 x 720` logical canvas
   to fit the window (max scale 1.6).
6. **Input**: keyboard (`keys` map), mouse steering and click-to-fire on the
   canvas, and touch (virtual joystick, FIRE and BOMB buttons).
7. **Game state and entities**: `resetGame()` initialises the state; `update(dt)`
   runs the simulation (bullets, enemies, enemy bullets, particles, power-ups,
   collisions); `updateBoss(dt)` drives boss patterns (`fire*` helpers);
   `draw()` renders everything; `loop(now)` is the `requestAnimationFrame` loop.
8. **Lifecycle**: `loseLife()`, `endGame()`, `togglePause()`, `killBoss()`.

## Game rules worth knowing

- **Difficulty** (`DIFFICULTIES`): easy, normal (default), hard. Each sets
  spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs,
  and a boss HP multiplier.
- **Stages**: a stage ends after `KILLS_PER_STAGE` kills, which triggers a boss
  warning and a boss spawn about 1.2 s later. Defeating the boss adds 500
  points, increments `stage`, and resets the stage kill count.
- **Enemy types** unlock by stage (`minStage`): grunt from stage 1,
  interceptor from stage 1 (weaving), tank from stage 2 (4 HP).
- **Bosses**: HP is `70 + stage * 35`, scaled by the boss design's `hpMul` and
  the difficulty's `bossHpMul`. Each boss has telegraphed attack patterns.
- **Weapons**: Vulcan and Laser, each with three power levels collected from
  weapon power-ups.
- **Power-ups** (`maybeSpawnPowerUp`): extra life (6%), bomb (8%), weapon (20%)
  drop chances on kills.
- **Bombs**: limited smart bombs (X or Shift on keyboard, BOMB button on
  touch). They clear the screen and damage bosses.
- **Scoring**: kills add points through a kill-chain multiplier (`chain`,
  `multiplier`, `breakChain`). The best score is saved on game over.

## Persistence keys

These `localStorage` keys are a public-ish contract, so do not rename them
without a migration:

| Key | Values |
| --- | --- |
| `skyfighter.bestScore` | number |
| `skyfighter.muted` | `'1'` or `'0'` |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; unset means follow the OS `prefers-reduced-motion` |

Every access must tolerate storage being unavailable (private mode, blocked
storage). Keep reads and writes simple and guarded.

## Controls (for reference when changing input code)

- Keyboard: arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- Mouse: move to steer, click to fire.
- Touch: drag left zone to fly, tap FIRE, tap BOMB.

## Conventions

- Keep the game a single self-contained file. Do not add external scripts,
  fonts, or assets unless the user asks; the page currently depends on none.
- Match the existing style: `const`/`let`, two-space indentation, small
  top-level helpers, and comments only where the logic is not obvious.
- Use `dt`-based movement for anything that should be frame-rate independent.
  Many values in the code are per-frame and were tuned against the existing
  loop, so check the surrounding code before changing timing.
- Reduced motion and audio mute are user-facing settings. New visual effects
  like shake or flashes should respect `reducedMotion` and the existing helpers.
- Performance: `trackPerf` lowers the `quality` level when frame times are
  poor. New particle or draw-heavy effects should consider that level.
- Keep the HUD and overlay usable at phone width; the viewport is non-scalable
  and touch-first.

## Git workflow

- Develop on the branch named in the session instructions, and push with
  `git push -u origin <branch>`.
- Write descriptive commit messages in the imperative mood, with a short
  summary line and a body for the reasoning when it is not obvious.
- Do not open a pull request unless the user asks for one.
