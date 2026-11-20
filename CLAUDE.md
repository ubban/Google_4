# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`, with inline CSS and one inline `<script>` block. There are no external assets, no build step, no package manager, and no network calls. Audio is synthesized with WebAudio.

Files:

- `index.html` - the whole game (about 1,670 lines).
- `README.md` - a one-line title. Keep it short.
- `CLAUDE.md` - this file.

## Running the game

There is no build or install step. Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server` from the repo root.

There is no test suite, linter, or CI configured. Verify changes by loading the page and playing it. For automated checks, a headless Chromium with Playwright works (see the environment notes for the pre-installed browser).

## Code layout inside `index.html`

The script is one IIFE. Sections are marked with `// ---------- Name ----------` comments, and they appear in this order:

| Section | Purpose |
| --- | --- |
| Storage keys (`STORAGE_*`) | `localStorage` keys: best score, mute, difficulty, reduced motion. |
| Adaptive performance (`trackPerf`) | Frame-time tracking. |
| `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` | Tuning tables. Change balance here first. |
| Audio (`ensureAudio`, `beep`, `noiseBurst`) | WebAudio synth. Audio starts only after a user gesture. |
| HUD and effects (`renderHearts`, `renderBombs`, `flashHit`, `showStageBanner`, `screenShake`) | DOM HUD and visual feedback. |
| Canvas sizing (`BASE_W = 480`, `BASE_H = 720`, `resize`) | Fixed logical resolution, scaled to fit. |
| Input | Keyboard, touch joystick, fire button, mouse steering. |
| Game state (`resetGame`, `update`, `draw`) | Core loop logic and rendering. |
| Boss patterns (`BOSS_PATTERNS`, `fire*`) | Bullet patterns for each boss. |
| `loop` | `requestAnimationFrame` loop. `dt` is clamped to 50 ms. |

## Controls

- Move: arrow keys or WASD (keyboard), a touch joystick (drag left side), or the mouse hovering over the canvas.
- Fire: `Space` (keyboard), hold the FIRE button (touch), or hold the mouse button.
- Smart bomb: `X` or Shift.
- Pause: `P` or `Escape`.

## Game design facts

- Difficulty: `easy`, `normal` (default), or `hard`. Each sets spawn rates, enemy fire chance, starting lives and bombs, and a boss HP multiplier.
- Stages: a stage ends after `KILLS_PER_STAGE` (18) kills, and then a boss fights. Six bosses are defined, in order: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN. Each has a `hpMul` and its own pattern function.
- Enemies: `grunt`, `interceptor`, and `tank` (see `ENEMY_TYPES`).
- Scoring uses a combo chain (`addScore`, `breakChain`) and a score multiplier.
- Accessibility: a reduced-motion setting, persisted in `localStorage`, defaults to the OS `prefers-reduced-motion` value and toggles a `reduced-motion` class on `<body>`.

## Conventions

- Keep the single-file structure unless the user asks to split it. Do not add a build step, a framework, or external dependencies without asking.
- Keep the 480x720 logical canvas. Put new gameplay numbers in the existing tuning tables instead of hard-coding them.
- Match the existing style: `const` and `let`, arrow functions for handlers, two-space indentation, and section banner comments.
- Storage keys use the `skyfighter.*` prefix. Existing `localStorage` reads and writes are not wrapped in try/catch, so they can throw in restricted browser modes. When you add storage access, wrap it in try/catch so the game still works without it.
- Do not add `package.json`, lockfiles, or generated output to the repo.

## Git

- Develop on the branch assigned for the session, not on the default branch.
- Commit messages are short imperative summaries. The existing history is `Initial commit` and `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.
- Do not open a pull request unless the user asks for one.
