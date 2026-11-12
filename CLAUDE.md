# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up
(Raiden-style) built on HTML5 canvas. There is no build step, no package manager,
and no test suite.

Files:

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript
  (one IIFE inside an inline `<script>`). Roughly 1,700 lines.
- `README.md` — a stub title. Keep it short; don't pad it.
- `CLAUDE.md` — this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server`) and visit the page. No install
step is needed.

## Code layout inside `index.html`

The script is organized into sections, roughly in this order:

1. **DOM references** — `const xxxEl = document.getElementById(...)` at the top.
2. **Storage keys** — `STORAGE_*` constants (`skyfighter.bestScore`,
   `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
3. **Adaptive performance** — `trackPerf` lowers/raises a `quality` level
   (0 full, 1 medium, 2 low) based on rolling frame time.
4. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage
   sky palettes), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here.
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is
   synthesized with WebAudio; there are no audio files.
6. **HUD and effects** — `renderHearts`, `renderBombs`, `showComboPop`,
   `showStageBanner`, `screenShake`, etc.
7. **Input** — keyboard (`keydown`/`keyup` on `window`), mouse on the canvas, and
   touch: a virtual joystick plus fire and bomb buttons.
8. **Game state** — `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`,
   `maybeSpawnPowerUp`, `addScore`, `useBomb`, `fireWeapon`, `applyPowerUp`.
9. **Bosses** — `BOSS_PATTERNS` (bullet-pattern functions), `updateBoss`,
   `killBoss`.
10. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
11. **Main loop** — `loop(now)` clamps `dt` to 50 ms and calls `update` then
    `draw` while the game is running and not paused.

Controls to keep in sync if you change them: arrows/WASD move, Space or hold to
fire, X / Shift for a bomb, P / Escape to pause. Touch and mouse have their own
handlers.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split.
  Don't add a bundler, framework, or npm dependencies without being asked.
- **Vanilla JS, ES2015+ style.** Match the existing style: `const`/`let`,
  arrow functions in callbacks, 2-space indentation, and single quotes in JS.
- **Persist only through the existing `STORAGE_*` keys** and wrap any new
  `localStorage` access in a way that tolerates it being unavailable (private
  browsing, blocked storage).
- **Accessibility.** Respect `prefers-reduced-motion` (see the `reducedMotion`
  flag and the `reduced-motion` body class). Announce important state changes
  through the `srLive` live region using `announce()`.
- **Mobile.** The layout must work at phone width and with touch input. Don't
  rely on hover or keyboard-only interactions for gameplay.
- **Performance.** The loop runs every frame. Avoid allocating large arrays or
  creating new objects per frame in hot paths, and respect the `quality` level
  when adding expensive effects.

## Making changes

- Balance tweaks (speeds, spawn rates, HP, scores, boss multipliers) belong in
  the tuning tables, not scattered through the logic.
- When adding an enemy, boss, or power-up, extend the existing table and handle
  it in `update`/`draw` the same way the current types are handled.
- After editing, open the page in a browser and play through at least one stage
  on desktop and, if the change touches layout or input, at a phone-sized
  viewport. There are no automated tests, so manual verification is the check.
- Keep commits focused. Don't reformat unrelated code.

## Git

- Development branch for this work: `claude/gifted-tesla-voij5y`.
- Commit messages: short imperative subject, optional body explaining the why.
- Don't open pull requests unless the user asks for one.
