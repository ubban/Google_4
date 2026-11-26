# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based Raiden-style vertical shoot-'em-up. There is no build system, package manager, test suite, or server dependency.

- `index.html` — the entire game: markup, CSS (`<style>`, roughly lines 10–361), and one inline IIFE in a `<script>` block (roughly lines 422–1668).
- `README.md` — placeholder; only a title and a timestamp line.

To run it, open `index.html` in a browser. No install or bundling step.

## Repository layout and conventions

- Keep the game self-contained in `index.html`. Do not add a bundler, framework, or external runtime dependency unless the user asks for one.
- Game code lives inside a single `(function () { 'use strict'; ... })();` IIFE. Don't introduce globals; expose nothing on `window` unless there is a clear need.
- Section headers in the script use `// ---------- Name ----------` comments. Follow that style for new sections.
- Constants are `UPPER_SNAKE_CASE` (e.g. `BASE_W`, `KILLS_PER_STAGE`, `STORAGE_BEST`). Config-style tables are plain object/array literals near the top of the script: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
- DOM element references are fetched once at the top of the IIFE with `document.getElementById`. Reuse those rather than re-querying.
- Persistence uses `localStorage` with keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Any new persisted setting should use the same prefix and be wrapped so that a blocked or unavailable `localStorage` does not break the game.

## Architecture notes

- **Fixed logical resolution:** the canvas is `480×720` (`BASE_W`/`BASE_H`). `resize()` scales the CSS size to fit the window (capped at 1.6×). All game coordinates are in logical pixels; don't size things from `window` dimensions.
- **Main loop:** a single `requestAnimationFrame` loop (`loop(now)`) calls `trackPerf`, then `update(dt)` and `draw()` while `running && !paused`. `dt` is clamped to 0.05 s so tab switches don't teleport entities.
- **Adaptive quality:** `trackPerf` watches a rolling 45-frame average and steps `quality` between 0 (full), 1 (medium), and 2 (low). Drawing code should respect this level when adding effects.
- **Difficulty:** `DIFFICULTIES` controls spawn rate, enemy speed, enemy fire chance, starting lives/bombs, and boss HP multiplier. Add tuning here rather than hard-coding numbers in `update`.
- **Stages:** reaching `KILLS_PER_STAGE` (18) kills in a stage triggers that stage's boss (`spawnBoss`). Defeating the boss (`killBoss`) increments `stage`. The stage selects the `THEMES` background (`THEMES[(stage - 1) % THEMES.length]`), and enemy types gate on `minStage`.
- **Bosses:** `BOSSES` holds visual and HP-multiplier data; `BOSS_PATTERNS` holds the bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireCross`, etc.) that `updateBoss` cycles through.
- **Audio:** all sound is synthesized with the Web Audio API (`beep`, `noiseBurst`, `sfx.*`). There are no audio asset files. `ensureAudio()` must run from a user gesture before audio plays.
- **Input:** keyboard (WASD/arrows to move, Space to fire, X or Shift for bomb, P or Escape to pause), mouse on the canvas, and on-screen touch controls (joystick, fire, bomb) for mobile. Keep all three working when changing controls.
- **Accessibility:** there is a `role="status"` live region (`#srLive`) fed by `announce()`, and a reduced-motion toggle that also respects `prefers-reduced-motion`. Screen shake and flash effects should be skipped or dampened when `reducedMotion` is on.

## Making changes

- Make the smallest change that does the job; this is a single file, so keep diffs focused.
- After editing, open `index.html` in a browser (or a headless browser) and verify the start, play, pause, game-over, and restart flows. There are no automated tests.
- Check touch layout at phone width as well as desktop.
- Don't commit generated or temporary files. Keep the working tree clean apart from the intended edits.

## Git

- Develop on the branch assigned to the session. Don't push to other branches without explicit permission.
- Commit messages should be short and imperative, describing the change (for example, "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
- Don't open pull requests unless the user asks for one.
