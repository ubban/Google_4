# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, static web project. It currently contains one game:

- **`index.html`** — *Sky Fighter*, a Raiden-style vertical shoot-'em-up. It is a single self-contained file: inline `<style>`, inline markup, and one inline `<script>` that drives a `<canvas>` game loop. There is no build step, no package manager, and no external dependencies.
- **`README.md`** — a stub (title only).

Treat `index.html` as the whole application unless the project grows. Do not introduce a framework, bundler, or npm dependency without the user asking for it.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` from the repo root and visit `http://localhost:8000`).
- There is no test suite, linter, or CI configured. Verify changes by playing the game in a browser: check the title/overlay, a full run, pause (P/Esc), bomb, boss fight, game over, and the mobile touch controls if the change touches input or layout.
- For UI or gameplay changes, prefer checking in a real browser (e.g. Playwright with the pre-installed Chromium) over reasoning from the code alone.

## Code layout inside `index.html`

- **Head/style block** — CSS custom look: dark page background, gradient sky on `#gameCanvas`, pill-style HUD, glassmorphic overlay card, mobile control layout.
- **Markup** — canvas, HUD (`#hud`, hearts, bombs, score), overlay card, pause/mute/motion buttons, difficulty buttons, touch controls (move zone, fire, bomb).
- **Script** (single IIFE starting around line 423):
  - Persistence keys: `STORAGE_*` constants (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`).
  - Tuning: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `BASE_W`/`BASE_H` (logical play-field size, 480×720).
  - Audio: a small WebAudio synth (`beep`, `noiseBurst`, `sfx.*`), created lazily via `ensureAudio()` on first user gesture.
  - Rendering helpers: `drawPlane`, `drawBossShip`, explosions, screen shake, HUD renderers.
  - Game logic: `update(dt)`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `resetGame`, `togglePause`.
  - Input: keyboard (`keys`, `keydown`/`keyup`), mouse steering on the canvas, virtual joystick + buttons for touch.

## Conventions

- **Keep it a single file.** New game code goes in the existing `<script>` IIFE unless there is a clear reason to split it out. If you do split, reference scripts with relative paths and keep them dependency-free.
- **Match the existing style:** 2-space indentation, `const`/`let` (no `var`), single quotes in JS, arrow functions for callbacks, and the existing `camelCase` naming. Keep the `STORAGE_*` key prefix `skyfighter.` for any new persisted value.
- **Logical coordinates.** Game objects are positioned in the `BASE_W` × `BASE_H` logical space; the canvas is scaled by `resize()`. Don't hard-code pixel sizes that bypass that scaling.
- **Time-based updates.** `update(dt)` takes a delta time; keep new movement, timers, and spawn rates frame-rate independent.
- **Respect the user's settings.** Honor the mute toggle (`muted`), the reduced-motion preference (`reducedMotion` — screen shake and flashes should be suppressed or softened), and the difficulty table for any new tunable.
- **Input parity.** Any new control must work on both keyboard/mouse and touch. Touch handlers call `preventDefault()` with `{ passive: false }` where needed to stop scrolling and zoom.
- **Audio gating.** Audio must only start after a user gesture; go through `ensureAudio()` rather than creating an `AudioContext` directly.
- **No external network calls** at runtime. Everything (sprites, sound, fonts) is generated in code or uses system font stacks.

## Git and delivery

- Default working branch for this session: `claude/gifted-tesla-7pvmzz`. Commit and push there; do not push to other branches without explicit permission.
- Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.
- Keep commits focused: one logical change per commit, with a short imperative subject line and a body that explains *why* when it isn't obvious.

## Things to avoid

- Don't add build tooling, `package.json`, or a framework unless asked.
- Don't commit generated files, screenshots, or Playwright traces into the repo; keep scratch output in the session scratchpad.
- Don't remove or rename the `skyfighter.*` localStorage keys — doing so resets players' saved best score, mute, difficulty, and motion settings.
