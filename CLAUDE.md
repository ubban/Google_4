# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- The entire game lives in a single file, `index.html` (about 58 KB): inline CSS in `<style>`, markup, and one IIFE-wrapped `<script>` that drives an HTML5 `<canvas>` (2D context).
- `README.md` is a placeholder and contains no setup or usage instructions.
- There is no build step, package manager, test suite, linter, or CI configuration.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture (inside `index.html`)

The script is one IIFE. Key pieces, roughly in file order:

- **DOM refs and storage keys**: element lookups at the top of the script. Persistent settings use `localStorage` under `skyfighter.*` keys (`bestScore`, `muted`, `difficulty`, `reducedMotion`). All storage access should stay defensive, since storage can be unavailable.
- **Tuning tables**:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed and shoot chance, lives, bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (18) controls stage progression.
  - `THEMES`: six sky palettes that cycle by stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocked at stage 2).
  - `BOSSES`: six boss designs with distinct colors and HP multipliers.
- **Audio**: Web Audio API synthesis only (`beep`, `noiseBurst`, and the `sfx` table). No audio files. The `AudioContext` is created lazily in `ensureAudio()` after a user gesture.
- **Entities and systems**: `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `useBomb`, `fireWeapon`, and the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) called from `updateBoss`.
- **Game loop**: `update(dt)` advances the simulation, `draw()` renders the frame, and `loop(now)` is the `requestAnimationFrame` driver. `endGame()` handles game over and saving the best score.
- **Input**: keyboard (`keys[e.code]`; arrows/WASD move, Space fires, X/Shift bombs, P/Esc pauses), mouse (steer and click to fire), and touch controls (joystick zone, FIRE and BOMB buttons).
- **Accessibility**: `prefers-reduced-motion` is honored and can be toggled. Screen shake, flash, and vignette pulse are gated by it. Announcements go to `#srLive` (an `aria-live` region).

## Conventions and guidance

- Keep everything in `index.html` unless there is a clear reason to split files. Do not add a bundler, framework, or npm dependencies without asking.
- Put gameplay numbers in the existing config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Keep the game loop frame-rate independent by using the `dt` value passed to `update`.
- Preserve mobile support: `touch-action: none`, the viewport meta, and the on-screen touch controls.
- Keep the layout working at phone widths. The HUD uses `clamp()` sizing and wraps.
- Since there are no automated tests, verify changes by loading the page in a browser and playing through at least one stage, on desktop and in a narrow viewport when UI changes.

## Git

- Development branch for this work: `claude/gifted-tesla-nl5q5x`. Push with `git push -u origin <branch>`.
- Write commit messages that describe the gameplay or UI change in the imperative mood.
