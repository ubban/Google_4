# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. Everything (markup, CSS, and JavaScript) lives in one file, `index.html` (~1,670 lines). `README.md` is essentially empty and has no setup instructions.

Tracked files:

- `index.html` — the complete game
- `README.md` — placeholder

## Running and testing

- There is no build step, package manager, bundler, or test suite. Do not add one unless asked.
- To run the game, open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- Verification is manual: load the page, start a game, and exercise the change. There are no automated tests to run.

## Code layout (inside `index.html`)

- **`<head>`**: meta tags (viewport locked to device width, dark `theme-color`), an inline SVG favicon, and a single inline `<style>` block. Colors and layout are hard-coded; there are no CSS variables.
- **`<body>`**: the `#container` wrapper, the `#gameCanvas` canvas, the HUD (`#hud`), the overlay card with the Start button (`#startBtn`), and on-screen touch controls (joystick zone, fire button, bomb button).
- **`<script>`** (starts around line 422): one `(function () { ... })()` wrapper holding all `const`/`let` state and functions. Key sections, roughly in file order:
  - Storage keys (`STORAGE_*`) and persisted settings: best score, mute, difficulty, reduced motion.
  - Audio: lazily created `AudioContext` via `ensureAudio()`, with `beep` and `noiseBurst` helpers and named `sfx.*` sounds.
  - Tuning tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. `KILLS_PER_STAGE` controls when a boss appears.
  - Input: keyboard handlers, a touch joystick, fire and bomb buttons, and mouse/pointer handling.
  - Game state: `resetGame()` initializes the run; `update(dt)` advances the simulation; `draw()` renders; `loop(now)` drives the `requestAnimationFrame` loop.
  - Weapons and power-ups: `fireWeapon()` (Vulcan spread and Laser lanes, 3 levels each), `applyPowerUp()`, `useBomb()`.
  - Bosses: `spawnBoss()`, `updateBoss()`, and the attack patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, and others).
  - Adaptive quality: `trackPerf(dt)` lowers visual quality (`quality` 0–2) when frame times are slow.
  - Accessibility: `announce()` writes to the `#srLive` live region. Reduced motion defaults to the `prefers-reduced-motion` media query and can be toggled by the user; it dampens screen shake and similar effects.

## Gameplay constants to know

- Base logical resolution: `BASE_W = 480`, `BASE_H = 720`. The canvas is scaled to fit by `resize()`.
- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. The default is `normal`.
- Controls: arrows or WASD to move; Space to fire; X or Shift to use a bomb; P or Escape to pause.
- The HUD is a pill-style display. Lives render as hearts and bombs as icons, both via `renderHearts` and `renderBombs`.

## Conventions

- Keep the game as a single self-contained file unless the user asks for a split. Do not introduce external dependencies or CDN links without a clear reason; the page currently works fully offline.
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const` for tuning tables, `let` for mutable state, and short descriptive function names.
- Put gameplay numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and so on) rather than scattering magic numbers through the update and draw code.
- Persist user settings only through the existing `STORAGE_*` keys. The current code reads and writes `localStorage` directly, so when touching that code, consider guarding it with try/catch so the game still runs when storage is blocked.
- Preserve mobile support: keep touch controls working, keep `touch-action` set as it is, and check layout at phone widths.
- Respect reduced motion (`reducedMotion`) when adding screen shake, flashes, or other motion effects.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-c0yxns`). Do not push to other branches without permission.
- Write clear commit messages that describe the gameplay or UI change. Do not commit unless asked.
- Do not open a pull request unless the user explicitly asks for one.

## Things to avoid

- Do not rewrite the whole file for small changes; edit the specific section instead.
- Do not remove the audio-context gating: `ensureAudio()` is called from user gestures because browsers block audio until the user interacts with the page.
- Do not commit generated or temporary files; the repo currently tracks only `index.html` and `README.md`.
