# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas app.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–362), canvas and HUD/overlay DOM, and one inline `<script>` IIFE (starts ~line 422) holding all game logic.
- `README.md` — a one-line title. There is no other documentation.
- No package manager, build step, test suite, linter, or CI config exists.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, e.g. `python3 -m http.server` and visit `http://localhost:8000`. Nothing needs to be installed.

## Code layout inside `index.html`

The script is one `(function () { 'use strict'; ... })();` block. Its sections, in order:

1. **DOM references** — `getElementById` lookups at the top of the IIFE.
2. **Constants** — `STORAGE_*` keys, `DIFFICULTIES` (easy/normal/hard tuning), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
3. **Persistence** — best score, mute, difficulty, and reduced-motion state, all in `localStorage` under the `skyfighter.*` prefix.
4. **Adaptive performance** — `trackPerf` lowers a `quality` level (0–2) when frame times are slow.
5. **Audio** — a lazily created `AudioContext` (`ensureAudio`) plus synthesized `beep` / `noiseBurst` SFX. No audio files.
6. **HUD and effects** — hearts, bombs, combo pops, stage banner, screen shake, flash.
7. **Input** — keyboard (`keys` map), mouse, touch joystick, and fire/bomb buttons.
8. **Game state** — `resetGame`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), scoring (`addScore`, `breakChain`), weapons (`fireWeapon`, `applyPowerUp`), bombs (`useBomb`).
9. **Bosses** — `BOSS_PATTERNS` are bullet-pattern functions (`fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`, …). `updateBoss` / `killBoss` drive them.
10. **Lifecycle** — `loseLife`, `endGame`, `togglePause`.
11. **Rendering** — `draw()` plus helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
12. **Main loop** — `update(dt)`, `draw()`, and `loop(now)` on `requestAnimationFrame`.

Gameplay is fixed at a `480×720` logical canvas (`BASE_W`, `BASE_H`), scaled by `resize()`.

## Controls (keep the HTML hint in sync)

The in-game hint is at `index.html` ~line 417. If you change a control, update that text too.

- Keyboard: arrows/WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag the left zone to fly, FIRE and BOMB buttons on the right.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a bundler, framework, or dependencies without asking.
- **No external assets.** Graphics are drawn with canvas primitives and audio is synthesized. Keep it that way. The favicon is an inline SVG data URI.
- **Match the existing style.** Two-space indentation, `const`/`let`, arrow functions for callbacks, and `camelCase` names. Tuning values live in the constant tables near the top of the script, so prefer changing those over hard-coding numbers in logic.
- **Persisted keys are part of the contract.** Renaming a `skyfighter.*` storage key silently resets the player's saved best score, mute, difficulty, or motion setting. Only rename one deliberately, and mention it in the commit.
- **Accessibility.** Respect `prefers-reduced-motion` (the `reduced-motion` body class and `reducedMotion` flag gate screen shake and similar effects). Keep the `srLive` live region updated for important announcements via `announce()`.
- **Performance.** The loop runs every frame. Avoid allocating in hot paths and keep the `quality` level in mind when adding particles or effects.

## Verifying changes

There are no automated tests. After a change:

1. Open `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage transition and one boss fight if the change touches combat, spawning, or bosses.
3. Check the mobile layout (narrow viewport or device emulation) if the change touches HUD, overlays, or touch controls.
4. Confirm that best score and settings still persist after a reload.

## Git workflow

- Develop on the branch the session names (currently `claude/gifted-tesla-kwz5a5`). Do not push to `main` directly.
- Write descriptive commit messages in the imperative mood. The existing history uses a short subject line plus a bulleted body.
- Do not open a pull request unless the user asks for one.
