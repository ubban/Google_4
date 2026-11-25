# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A small static web project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser.

- `index.html` — the whole game: inline `<style>`, inline markup, and one inline `<script>` (an IIFE). No external assets, libraries, or network calls.
- `README.md` — a one-line title. There is no other documentation.

There is no build step, no package manager, no test suite, and no linter configured.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

To verify a change, load the page in a browser and play through the relevant path. Syntax-only checks are not enough for gameplay changes.

## Code layout inside `index.html`

The script is one IIFE. Roughly in order:

1. **DOM references** — `document.getElementById` lookups at the top of the script.
2. **Constants** — `STORAGE_*` keys for `localStorage`, `DIFFICULTIES` (spawn rates, lives, bombs, boss HP multiplier per difficulty), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (name and colour scheme, HP multiplier).
3. **Persisted settings** — best score, mute, difficulty, and reduced-motion, all read from and written to `localStorage`.
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`), `beep`/`noiseBurst` primitives, and the `sfx` table of named sound effects.
5. **HUD and feedback** — hearts/bombs rendering, combo pop, stage banner, screen shake, and flash effects.
6. **Input** — keyboard (`keys`), mouse, touch joystick and buttons. Touch is detected via `isTouchDevice`.
7. **Canvas sizing** — `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the window (capped at 1.6x).
8. **Game state and logic** — `resetGame`, `spawnEnemy`, `spawnBoss`, `update(dt)`, `updateBoss`, weapon firing (`fireWeapon` and the boss pattern helpers `fireRadial`, `fireCross`, etc.), `applyPowerUp`, `useBomb`, `loseLife`, `killBoss`, `endGame`.
9. **Rendering** — `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
10. **Main loop and startup** — `loop(now)` runs `update` and `draw` while `running && !paused`, with `dt` clamped to 50 ms. `trackPerf` adjusts a `quality` level (0 full, 1 medium, 2 low) based on frame times. The loop starts with `requestAnimationFrame(loop)`.

Game-balance numbers live in the constant tables above. Change those tables rather than hard-coding values in the logic where possible.

## Conventions

- **Keep it a single file.** Do not split `index.html` into modules or add a build step unless the user asks for it.
- **Match the existing style:** two-space indent, single quotes in JS, `const`/`let`, camelCase identifiers, and `UPPER_SNAKE_CASE` for top-level constant tables.
- **No new dependencies or external URLs** without asking. The game currently works fully offline.
- **Respect the accessibility features.** Reduced motion (`reducedMotion`, which also honours `prefers-reduced-motion`) gates screen shake, flashes, and pulses. New visual effects should go through the same check. Announcements for screen readers go through `announce()` into the `srLive` live region.
- **Keep the mobile layout working.** The page must work at phone width with touch controls (`#touchControls`). Test touch changes with touch emulation.
- **Persisted keys** use the `skyfighter.*` prefix. The current `localStorage` reads and writes are not wrapped in try/catch; new storage access should be, so the game still runs if storage is unavailable or throws.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-wvqbz8`). Do not push to other branches without permission.
- Do not open a pull request unless explicitly asked.
