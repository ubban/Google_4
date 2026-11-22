# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a single `<canvas>`.

- `index.html` (~1,670 lines) — the entire game: inline CSS (`<style>`, ~lines 10–362), the canvas and HUD markup, and one inline `<script>` IIFE (~lines 422–1667) containing all game logic, rendering, audio, and input.
- `README.md` — a one-line placeholder (no real documentation yet).

There is no build system, no package manager, no test suite, and no dependencies. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify changes by loading the page and playing; there are no automated tests or linters configured.
- Browser automation (Playwright) is available in the remote environment if a headless check is needed.

## Code structure (inside `index.html`)

The script is a single IIFE with `'use strict'`. Major sections, in source order:

1. **DOM references and constants** — element lookups by ID, `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`).
2. **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0–2) up or down.
3. **Tuning tables** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six background palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses: VANGUARD → TITAN).
4. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and an `sfx` object built on the Web Audio API. Audio is created lazily on first user gesture.
5. **HUD and feedback** — hearts/bombs rendering, hit flash, combo pop, stage banner, screen shake, mute/motion/difficulty toggles.
6. **Layout and input** — `resize` (fixed logical size `BASE_W=480 × BASE_H=720`, scaled to fit), keyboard handling, and a virtual joystick plus fire/bomb buttons for touch devices.
7. **Game state** — `resetGame`, and arrays for `bullets`, enemies, enemy bullets, `powerUps`, and explosions.
8. **Spawning and combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, scoring (`addScore`, multiplier chain via `breakChain`).
9. **Update** — `update(dt)` moves entities, resolves collisions (`rectsOverlap`), and advances stages.
10. **Power-ups** — `applyPowerUp` handles `life`, `bomb`, and `weapon` (weapon levels 1–3, toggling between `vulcan` and `laser` after level 3).
11. **Bosses** — `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring) selected by `updateBoss`; `killBoss`.
12. **Lifecycle** — `loseLife`, `endGame`, `togglePause`.
13. **Rendering** — `draw`, `drawPlane`, `drawBossShip`, `drawClouds`.
14. **Main loop** — `loop(now)` caps `dt` at 50 ms, calls `update` and `draw` while running and not paused, then re-queues via `requestAnimationFrame`.

Game state lives in module-level `let` variables within the IIFE; there is no global state to share across files because there is only one file.

## Controls

- **Keyboard:** arrow keys / WASD to move, Space to fire, X or Shift to use a bomb, P or Escape to pause.
- **Touch:** on-screen joystick (left), fire and bomb buttons (shown only on touch devices).
- **Persistent settings:** mute, reduced motion (defaults to the OS `prefers-reduced-motion` setting), and difficulty are saved to localStorage.

## Conventions

- Keep the game in a single self-contained file unless the user asks to split it. Do not add a bundler or framework.
- Keep all tuning values in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than hard-coding magic numbers in logic.
- Follow the existing style: 2-space indent, `const`/`let`, arrow functions for callbacks, section comments with `// ---------- Name ----------`.
- Preserve accessibility hooks when editing UI: the `srLive` ARIA live region (use `announce()`), and respect `reducedMotion` for screen shake and flashes.
- Persist only through the existing `STORAGE_*` keys, and wrap new localStorage access so it still works when storage is unavailable.
- Keep the layout working at phone width; the canvas scales to the viewport.

## Git

- Default development branch for this work: `claude/gifted-tesla-j35466`. Do not push to other branches without explicit permission.
- Do not create pull requests unless the user explicitly asks.
