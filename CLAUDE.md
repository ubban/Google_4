# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up (Raiden-style). There is no build step, no package manager, no framework, and no test suite.

- `index.html` — the entire game: inline `<style>`, markup for the HUD/overlays/touch controls, and an inline `<script>` that runs the game loop.
- `README.md` — minimal project readme.

Git: default working branch for this session is `claude/gifted-tesla-gjy0ee`.

## Running the game

No build is required. Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.

There are no automated tests or linters configured. Verify changes by loading the page and playing; for scripted checks, Playwright (Chromium) is available in the cloud environment.

## Code layout inside `index.html`

The script is one IIFE-style block, roughly in this order:

1. **DOM references** (`canvas`, `overlay`, HUD elements, touch controls, etc.).
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. All `localStorage` access should stay wrapped so the game still works when storage is unavailable (private mode, blocked site data).
3. **Performance scaling** — `quality` (0 full / 1 medium / 2 low) adjusted by `trackPerf(dt)` based on frame times.
4. **Config tables**
   - `DIFFICULTIES` — `easy` / `normal` / `hard` tune spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
   - `KILLS_PER_STAGE` (18) — kills needed to advance a stage.
   - `THEMES` — six sky palettes, cycled per stage.
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocks at stage 2).
   - `BOSSES` — six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with colors and an `hpMul`.
5. **Audio** — Web Audio API synthesis (`beep`, `noiseBurst`, `sfx` table). Audio context is created lazily by `ensureAudio()` on first user gesture.
6. **Input** — keyboard (`keys` map by `KeyboardEvent.code`), touch joystick + fire/bomb buttons, and mouse on the canvas.
7. **Game state and logic** — `update(dt)` (entities, collisions, spawning, scoring) and `updateBoss(dt)`.
8. **Rendering** — `draw()`, plus helpers `drawPlane`, `drawBossShip`, `drawClouds`.
9. **Main loop** — `loop(now)` driven by `requestAnimationFrame`.

## Controls (for reference when testing)

- Move: arrow keys / WASD (keyboard), joystick (touch), mouse (canvas).
- Fire: hold Space / fire button.
- Bomb: `X` or Shift, or the bomb button.
- Pause: `P` or `Escape`.
- Mute and reduced-motion toggles are in the HUD/overlay.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a split. Do not introduce a build tool or dependencies without discussing it first.
- Keep it working on both desktop and phone-sized viewports (touch controls are shown when `ontouchstart` or `maxTouchPoints` is available). Don't add horizontal scroll.
- Respect `prefers-reduced-motion` and the in-game reduced-motion toggle when adding visual effects.
- Any new user-facing state that should persist belongs under the `skyfighter.*` localStorage namespace and must tolerate storage failures.
- Use `const`/`let`, plain ES2015+ JavaScript, and match the existing 2-space indentation and style.
- Tune gameplay numbers in the config tables rather than hard-coding them in logic.

## Keeping this file current

Update this CLAUDE.md when the structure changes materially (e.g. new files, a split of `index.html`, a build step, tests, or new storage keys or controls).
