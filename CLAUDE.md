# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in the browser on a single `<canvas>`.

## Layout

- `index.html` — the entire game: markup, inline `<style>`, and one inline `<script>`
  (~1,670 lines). There is no build step, bundler, or package manager.
- `README.md` — a one-line project title. Keep it short unless asked to expand it.
- `.git/` — standard git history on `origin` (`github.com/ubban/Google_4`).

There is no `package.json`, no test suite, no linter config, and no CI config in the repo.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server
  (e.g. `python3 -m http.server`). No install step is needed.
- Nothing is compiled or transpiled, so edits take effect on reload.
- No automated tests exist. Verify behavior by playing the game in a browser. For
  headless checks, Playwright with the pre-installed Chromium works.

## Code structure inside `index.html`

The script is one IIFE with `'use strict'`. Key sections, roughly in file order:

- **DOM references** — `getElementById` lookups near the top of the script.
- **Storage keys** — `skyfighter.*` in `localStorage` for best score, mute, difficulty,
  and reduced motion. Every `localStorage` access should be wrapped so the game still
  works when storage is unavailable (private mode, blocked site data).
- **Tuning tables** — `DIFFICULTIES` (spawn rates, enemy speed, shot chance, lives,
  bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes),
  `ENEMY_TYPES`, and `BOSSES`. Balance changes usually belong here, not in the update logic.
- **Audio** — a small WebAudio synth (`beep`, `noiseBurst`, `sfx.*`). Audio is created
  lazily through `ensureAudio()`, which is called from a user gesture.
- **Input** — keyboard (`keys[e.code]`), a touch joystick plus fire/bomb buttons, and
  mouse on the canvas. Touch controls are shown for touch devices.
- **Game state and loop** — `resetGame()`, `update(dt)`, `draw()`, and a single
  `requestAnimationFrame` loop that clamps `dt` to 50 ms and skips updates while paused
  or not running.
- **Boss patterns** — `BOSS_PATTERNS` plus the `fireRadial` / `fireAimedSpread` helpers.
- **Accessibility** — an `aria-live` region (`announce()`), a reduced-motion toggle
  (`body.reduced-motion`), and screen-shake that respects that setting.
- **Performance** — `trackPerf(dt)` for frame timing.

## Conventions

- Keep the game in a single self-contained `index.html` unless the user asks for a split.
  Do not add a build system, framework, or external dependencies without asking.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for
  callbacks, and short helper functions named in camelCase.
- Keep the UI usable at phone width: the page uses `touch-action: none`, safe-area insets,
  and a fixed full-viewport container. Test touch and keyboard paths when changing input.
- Preserve the accessibility hooks (`announce`, reduced motion) when touching HUD or
  effects code.
- Do not commit generated or local files. Keep changes focused on the request.

## Git

- Default development branch for this workspace: `claude/gifted-tesla-171pua`.
- Do not open pull requests unless the user explicitly asks for one.
