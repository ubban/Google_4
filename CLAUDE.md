# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game is one self-contained file, `index.html` (about 1,670 lines), with inline CSS and JavaScript and no build step.

| File | Purpose |
|------|---------|
| `index.html` | The whole game: markup, `<style>` block, and an IIFE `<script>` containing game logic, rendering, audio, and input |
| `README.md` | One-line placeholder; no real project docs yet |

Git history: `Initial commit`, then `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.

## Running and testing

- **No build, package manager, or test suite.** There is no `package.json`, bundler, linter, or test runner.
- To run the game, open `index.html` in a browser, or serve the directory with any static server (for example `python3 -m http.server`) and visit it.
- Verifying a change means loading the page and playing it. Headless Playwright works for this: launch Chromium and load `file:///.../index.html` or the served URL.
- Keep the game working as a single file. Do not split it into modules or add a build step unless the user asks for that.

## Architecture of `index.html`

Everything runs inside one `(function () { 'use strict'; ... })();` IIFE. Its main parts, in file order:

1. **Persistence keys**: `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every read and write is a plain call with no try/catch, so keep that in mind if you change storage.
2. **Configuration tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed and fire chance, starting `lives` and `bombs`, and `bossHpMul`.
   - `KILLS_PER_STAGE = 18`: kills needed before a stage boss warning.
   - `THEMES`: six sky palettes that cycle with the stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (4 HP, from stage 2).
   - `BOSSES`: six boss designs with colors and `hpMul`.
   - `BOSS_PATTERNS`: bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).
3. **Audio**: `ensureAudio()` lazily creates the Web Audio context, which browsers require to start after a user gesture. Sound effects come from `beep()` and `noiseBurst()`. Mute is persisted.
4. **UI helpers**: HUD hearts and bombs, flash and vignette effects, combo pops, stage banners, screen shake, and `announce()` for screen-reader live text.
5. **Input**: keyboard (arrows/WASD, Space to fire, X or Shift for bomb, P or Esc to pause), mouse (move to steer, click to fire), and touch (a left-side joystick, plus FIRE and BOMB buttons).
6. **Game state and loop**: `resetGame()` initializes state. `update(dt)` advances the simulation, including spawning, collisions, power-ups, and bosses. `draw()` renders everything to the canvas. `loop(now)` drives both through `requestAnimationFrame`.
7. **Power-ups**: `life`, `bomb`, and `weapon` (weapon level and type come from `applyPowerUp`).
8. **Canvas sizing**: a logical `BASE_W = 480` by `BASE_H = 720` playfield, scaled to fit the window by `resize()`.

## Conventions

- Keep code as plain ES2015+ JavaScript with no dependencies. Don't add libraries from a CDN unless asked.
- Keep styling in the `<style>` block and logic in the single `<script>`.
- Preserve the existing accessibility and mobile support: `prefers-reduced-motion`-style toggle (`reducedMotion`), `#srLive` live region, `aria` titles on icon buttons, touch controls, and `viewport-fit=cover` safe-area handling.
- Game balance lives in the constant tables near the top of the script. Prefer changing those over hard-coding numbers in the update loop.
- Use the existing naming: `STORAGE_*` for storage keys, `UPPER_CASE` for constant tables, and camelCase for functions and state.

## Git

- Development branch for this work: `claude/gifted-tesla-e3agh7`. Commit and push there.
- Commit messages should be short and imperative, in the style of `Add Sky Fighter: ...`.
