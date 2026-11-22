# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-file HTML5 canvas game.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript in one file (~1,670 lines). |
| `README.md` | Placeholder title only. |
| `CLAUDE.md` | This file. |

There is no build system, no package manager, no test suite, and no dependencies. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Architecture of `index.html`

The file is organized in three sections: a `<style>` block (~lines 10–360), the DOM in `<div id="container">` (~line 362 onward), and one `<script>` block (starts ~line 422). All script code sits inside one `(function () { 'use strict'; ... })();` wrapper; there are no modules.

Key parts of the script:

- **Persistence** – `localStorage` keys, all prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes are direct calls, so keep them consistent if you change them.
- **Configuration tables** (top-level `const`s, easy to tune):
  - `DIFFICULTIES` – `easy` / `normal` / `hard`, controlling spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` – kills required before a boss spawns (currently 18).
  - `THEMES` – six sky palettes, cycled per stage.
  - `ENEMY_TYPES` – `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
  - `BOSSES` – six boss designs, selected by `(stage - 1) % BOSSES.length`.
  - `BOSS_PATTERNS` – one bullet-pattern set per boss, built from helpers such as `fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`.
- **Main loop** – `requestAnimationFrame(loop)` calls `update(dt)` then `draw()`. A simple adaptive `quality` level is adjusted from frame-time measurements in `trackPerf`.
- **Audio** – Web Audio API, created lazily in `ensureAudio()` (required by browser autoplay rules). Sound effects are generated procedurally with `beep()` and `noiseBurst()`; there are no audio files.
- **Input** – keyboard (`keys[e.code]`), mouse (steer and click to fire), touch (virtual joystick plus FIRE and BOMB buttons). Canvas coordinates are mapped through `canvasPointFromEvent`.
- **Rendering** – all sprites are drawn with canvas primitives (`drawPlane`, `drawBossShip`); there are no image assets.
- **Game state** – plain `let` variables reset by `resetGame()`. Game over goes through `endGame()`, which updates the best score.

## Conventions

- **Keep it a single file.** The project is deliberately dependency-free. Do not introduce a bundler, framework, or external CDN script unless the user asks for it.
- **Preserve the style of the existing code.** It uses 2-space indentation, single quotes in JS, and `const` for tables and config.
- **Tune gameplay through the config tables** rather than hard-coding numbers in the update loop.
- **Respect accessibility settings.** The `reducedMotion` toggle gates screen shake, flashes, and pulses. New visual effects that move the screen or flash should check `reducedMotion`.
- **Keep touch and keyboard working.** The game targets both desktop and mobile. Test controls on both when changing input code.
- **Mind the mobile layout.** The HUD and overlay use `clamp()` and safe-area insets. Verify layout at phone widths after UI changes.

## Verification

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser (or use the pre-installed Playwright/Chromium if running headless).
2. Start a game, then check the affected behavior: shooting, bombs, stage transitions, boss fights, pause, game over, and best-score persistence.
3. Check the three difficulty levels if the change touches balance.
4. Check the mobile layout and touch controls if the change touches UI or input.

Report what you actually ran. If a check was not performed, say so.

## Git workflow

- Development branch for this work: `claude/gifted-tesla-cip8ez`.
- Commit messages follow the existing style: a short imperative summary line, optionally followed by a bullet list of details.
- Do not open a pull request unless explicitly asked.
