# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file: `index.html` (~1,670 lines) with inline CSS and one IIFE-wrapped `<script>` block. There is no framework, no bundler, no package manager, and no test suite. `README.md` is a placeholder.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No build step.
- **Tests:** none exist. Verify changes by playing the game in a browser; for scripted checks, use the pre-installed Playwright/Chromium described in the environment notes.
- **Lint/format:** none configured. Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, semicolons at statement ends.

## Architecture (inside `index.html`)

Sections of the `<script>` block, in order:

1. **DOM refs and storage keys** – `STORAGE_*` constants use the `skyfighter.*` prefix in `localStorage` (best score, mute, difficulty, reduced motion).
2. **Adaptive quality** – `trackPerf()` steps a `quality` level (0–2) based on a rolling frame-time average.
3. **Config tables** – `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six stage skies), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled per stage), `BOSS_PATTERNS`.
4. **Audio** – Web Audio synthesis via `beep()` / `noiseBurst()` and the `sfx` object; `ensureAudio()` must run after a user gesture.
5. **Resize** – logical canvas is fixed at **480×720**; it is scaled to fit the window (capped at 1.6×).
6. **Input** – keyboard (arrows/WASD move, Space fires, X/Shift bomb, P/Esc pause), touch joystick and fire/bomb buttons, and mouse on the canvas.
7. **Game state and logic** – `resetGame()`, `update(dt)`, weapons (`fireWeapon`, `applyPowerUp`), bombs (`useBomb`), scoring with combo chain (`addScore`, `breakChain`), stage progression, boss fights (`spawnBoss`, `updateBoss`, `killBoss`), `loseLife`, `endGame`, `togglePause`.
8. **Rendering** – `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`), plus HUD updates (hearts, bombs, weapon label, boss bar).
9. **Loop** – `loop(now)` clamps `dt` to 0.05s and runs `update` + `draw` only when running and not paused.

### Game rules to preserve

- Weapons: **vulcan** (spread shot) and **laser** (piercing). Power-ups upgrade the current weapon to level 3; after level 3 they toggle to the other weapon and reset to level 1.
- Power-up types: `life`, `bomb`, `weapon`.
- Each stage ends after `KILLS_PER_STAGE` kills, followed by a boss.
- Difficulty is set from `DIFFICULTIES`; `resetGame()` applies it to lives and bombs.
- Best score is persisted to `localStorage` under `skyfighter.bestScore`.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add dependencies or a build step without discussion.
- Remember that the file is shared by desktop and touch: any new input must work with both the keyboard/mouse and the touch controls.
- Respect the reduced-motion toggle when adding screen shake, flashes, or other motion effects (`reducedMotion`, `screenShake`).
- Keep audio and UI announcements accessible: use the existing `announce()` helper for screen-reader text.
- Do not use `localStorage` in new code without a try/catch; the existing code assumes it is available.

## Git

- Develop on the branch designated for this session; do not push to other branches without permission.
- Do not open pull requests unless explicitly asked.
