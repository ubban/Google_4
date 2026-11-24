# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~58 KB).
- `README.md` — a one-line placeholder title.
- No build system, package manager, bundler, test suite, or linter is configured.

## Running the game

There is nothing to install or compile. Open `index.html` directly in a browser, or serve the directory with any static file server, for example:

```
python3 -m http.server 8000
```

Then visit `http://localhost:8000/index.html`.

## Code layout inside `index.html`

- **`<head>`**: meta tags (viewport locked for mobile, `theme-color` `#0b1d3a`), the title, and an inline SVG favicon as a data URI. Keep the favicon and theme color consistent with each other.
- **`<style>`**: dark-only styling (`color-scheme: dark`). Includes the full-screen `#container`, the `#gameCanvas`, the pill-style `#hud`, the glassmorphic overlay card, and mobile touch controls.
- **`<body>`**: the HUD, a shared `#overlay` card (title, controls hint, difficulty selection, start/pause/game-over content), touch controls, and a single `<canvas id="gameCanvas">`.
- **`<script>`**: one IIFE (`(function () { ... })()`) so nothing leaks onto `window`. Within it, roughly in order:
  1. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (all `localStorage`).
  2. **Accessibility helpers** — `announce()` (screen-reader live region) and `trackPerf()`.
  3. **Tunable data tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  4. **Audio** — a lazily created Web Audio context (`ensureAudio()`), with `beep()`, `noiseBurst()`, and an `sfx` object.
  5. **HUD and effects** — hearts/bombs rendering, hit and bomb flashes, combo pops, stage banners, screen shake.
  6. **Input** — keyboard (`keys` map), touch joystick and fire/bomb buttons, and mouse aiming on the canvas.
  7. **Game state and logic** — `resetGame()`, `update(dt)`, spawning, collisions, power-ups, bombs, weapons, and boss patterns (`BOSS_PATTERNS`, `updateBoss`).
  8. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
  9. **Main loop** — `loop(now)`, driven by `requestAnimationFrame`, with `startBtn` wiring at the end.

## Game rules and controls (for reference when changing behavior)

- **Keyboard**: Arrow keys or WASD move via `keys`; `Space` fires; `X` / `Shift` uses a bomb; `P` / `Escape` pauses.
- **Mouse**: move the pointer over the canvas to steer; click (or hold the fire button) to fire.
- **Touch**: drag on the left side to fly (joystick), tap FIRE, tap BOMB.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rates, enemy speed, enemy shot chance, starting lives and bombs, and boss HP multiplier.
- **Stages**: each stage requires `KILLS_PER_STAGE` (18) kills, then a boss fight. Boss names come from `BOSSES`, and stage themes cycle through `THEMES` with `(stage - 1) % THEMES.length`.
- **Power-ups**: `life`, `bomb`, and `weapon` (weapon level). Handled in `applyPowerUp()`.
- **Score**: kill-chain multiplier (`addScore`, `breakChain`).
- **Best score** persists in `localStorage` and is updated in `endGame()`.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. If files are split, update this section and make sure relative paths still work when opened from disk.
- Prefer changing the existing data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) over hard-coding new magic numbers in logic.
- Use the existing `STORAGE_*` constants for any new persisted setting, and wrap new `localStorage` access so the game still works when storage is unavailable.
- Preserve accessibility hooks: announce important state changes through `announce()`, and respect the reduced-motion setting when adding screen shake or flashes.
- The UI must work on phones: keep `touch-action` handling and the touch controls intact, and test layout changes at narrow widths.
- Use plain JavaScript, HTML, and CSS. Do not add dependencies or a build step without a discussion first.
- Keep the code style of the file: 2-space indentation, single quotes in JS, and short comments only where the logic is not obvious.

## Verifying changes

There is no automated test suite. Before considering a change done:

1. Load `index.html` in a browser and check the console for errors.
2. Start a game on each difficulty and confirm enemies, bombs, power-ups, pause, and game over still work.
3. Check the HUD and overlays at both desktop and phone widths.
4. If a stage or boss is touched, play through to a boss fight and confirm stage progression.

## Git

- Development branch for this work: `claude/gifted-tesla-k4b3nx`.
- Commit messages should be short and descriptive, in the imperative mood, with a longer body when the change is non-trivial.
- Do not commit local editor or OS files, and do not commit secrets.
