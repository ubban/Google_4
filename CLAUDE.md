# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo (`ubban/google_4`) is a small, dependency-free web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-em-up, implemented as a single-page HTML5 canvas app.

| Path        | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–362), and JS (`<script>`, ~lines 422 onward). |
| `README.md` | Minimal placeholder (title only).                                       |

There is no build step, no package manager, no bundler, no test suite, and no linter configuration. Do not add these unless the user asks.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests. Verify changes by loading the page and playing it. For UI changes, check both desktop (keyboard) and mobile (touch) layouts.
- A Playwright browser is preinstalled in the cloud environment; use it for headless checks if needed.

## Architecture of `index.html`

The game lives in one IIFE inside the `<script>` tag. Key areas, roughly in file order:

- **Config tables** (~lines 485–527): `DIFFICULTIES` (spawn rates, enemy speed and fire chance, starting lives/bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six boss designs, cycled).
- **Persistence**: `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. All storage access should keep working if storage is unavailable; keep the key names stable so saved player data survives.
- **Audio**: Web Audio synthesis only (`beep`, `noiseBurst`, `ensureAudio`). No audio files. Audio is created lazily on first user gesture.
- **Layout/resize**: fixed logical canvas size `BASE_W × BASE_H` (480 × 720) scaled by `resize()`.
- **Input**: keyboard (`keys[e.code]`), touch joystick plus fire/bomb buttons, and mouse/pointer aiming on the canvas. Input is read per frame in `update()`.
- **Game state and loop**: `resetGame()` initializes state; `update(dt)` advances simulation; `draw()` renders; `loop(now)` drives both via `requestAnimationFrame`.
- **Weapons and power-ups**: `fireWeapon()` (Vulcan/Laser, 3 levels), `useBomb()` (clears screen, damages bosses), and `applyPowerUp()` for `life`, `bomb`, and `weapon` drops.
- **Bosses**: `spawnBoss()`, `updateBoss()`, `killBoss()`, and the attack-pattern library `BOSS_PATTERNS` (radial, spread, spiral, cross, homing, etc.).
- **Adaptive quality**: `trackPerf()` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to keep frame rate up. Rendering decisions should respect this level.
- **Accessibility**: a `reducedMotion` mode (user toggle, defaults to `prefers-reduced-motion`) disables screen shake and similar effects; `announce()` writes to a live region for screen readers.

## Conventions

- Keep everything in `index.html` unless there is a concrete reason to split files. The single-file, no-dependency design is intentional.
- Match the existing style: 2-space indentation, `const`/`let`, plain functions inside the IIFE, and `camelCase` names. Constants that are configuration use `UPPER_SNAKE_CASE`.
- Don't introduce external CDN libraries or frameworks. Keep the game runnable offline by opening the file.
- Keep changes focused. Don't reformat unrelated sections of the file, since it is one large file and diffs should stay reviewable.
- Preserve touch support and keyboard support together when changing controls.
- When tuning gameplay, adjust the data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) before changing logic.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-xe22wr`). Do not push to other branches without explicit permission.
- Do not create pull requests unless the user explicitly asks.
- Commit messages should describe the gameplay or code change in the imperative mood, with a short subject line and a body when the change is non-trivial.
