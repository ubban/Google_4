# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single static page with no build step, no package manager, and no test suite.

Files:

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` IIFE (roughly 1,670 lines).
- `README.md` — minimal placeholder.
- `CLAUDE.md` — this file.

To run it, open `index.html` in a browser, or serve the directory with any static server (for example `python3 -m http.server`).

## Architecture of `index.html`

- **Layout / CSS** (top of file): dark theme set via `color-scheme: dark`. The `#container` centers a fixed-aspect `#gameCanvas`. HUD elements (`#hud`, hearts, bombs, weapon and multiplier stats), the boss bar (`#bossBarWrap`), the stage banner, the vignette, the overlay menu, and the touch controls (`#touchControls`) are absolutely positioned on top of the canvas.
- **Constants** (script, around lines 463–516):
  - `STORAGE_*` keys in `localStorage` under the `skyfighter.*` namespace: best score, mute, difficulty, reduced motion.
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed and shoot chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE = 18`.
  - `THEMES`: six sky palettes, cycled per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, appears from stage 2).
  - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with a hull palette and `hpMul`. `BOSS_PATTERNS` (around line 1372) defines their attack patterns.
- **Audio**: a Web Audio `AudioContext`, created lazily by `ensureAudio()` on the first user gesture. Sound effects are built from `beep()` and `noiseBurst()`, exposed through the `sfx` object. Respect the `muted` flag.
- **Game state**: module-level `let` variables (`score`, `lives`, `bombCount`, `stage`, `kills`, `chain`, `multiplier`, and so on). `resetGame()` initializes them.
- **Main loop**: `requestAnimationFrame(loop)` → `update(dt)` for logic and `draw()` for rendering. `trackPerf(dt)` adjusts a `quality` level (0 full, 1 medium, 2 low) when frame times slip.
- **Input**: keyboard (`keys` map, keyed by `e.code`), mouse (canvas `mousemove` / `mousedown`), and touch (left-side joystick via `joystickStart/Move/End`, plus FIRE and BOMB buttons).
- **Controls**: arrows or WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses.
- **Power-ups** (`applyPowerUp`): `life`, `bomb`, and `weapon`. Weapons are `vulcan` and `laser` (`player.weapon`, shown by `weaponLabel()`).

## Conventions

- Keep the game in one file unless there is a clear reason to split it. Adding a build step or dependencies is a significant change; ask first.
- Do not add external CDN scripts or fonts. The page is self-contained.
- Read and write `localStorage` only through the existing `STORAGE_*` constants, and guard against it being unavailable.
- Reuse the existing `sfx` helpers for sounds rather than creating new audio contexts.
- Keep the mobile layout working: the viewport meta disables zoom, `touch-action: none` is set, and HUD sizes use `clamp()`. Check the touch controls after UI changes.
- Respect `reducedMotion` (the `body.reduced-motion` class) for any new animation.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, and short section comments.

## Verifying changes

There is no automated test or lint setup. Verify by:

1. Opening `index.html` in a browser (or serving the directory) and checking the browser console for errors.
2. Playing through a stage: movement, firing, a bomb, a stage transition, and a boss fight.
3. Checking pause (P / Esc), mute, difficulty switching, and the best-score persistence across reloads.
4. For layout changes, checking a narrow (phone-width) viewport.

If you add a test harness or tooling, document the commands here.

## Git

- Development branch for this work: `claude/gifted-tesla-y05ww1`. Push with `git push -u origin <branch>`.
- Commit messages should be short and descriptive, in the imperative mood (for example, "Add boss telegraph warning").
- Do not open a pull request unless explicitly asked.
