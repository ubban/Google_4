# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-file browser game, **Sky Fighter** (a Raiden-style vertical shoot-em-up). There is no build step, package manager, or test suite.

- `index.html` — the entire game: markup (`<body>` HUD, overlay, touch controls), inline CSS (`<style>`, before the body), and inline JavaScript (one `<script>` IIFE at the bottom of the file, ~lines 422–1667).
- `README.md` — minimal project title only.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing in a browser: check the start overlay, difficulty buttons, a full stage progression to a boss, pause/resume, game over, and the best-score update.
- For mobile-layout or touch changes, test with a touch emulator at phone width.

## Code layout inside the script

The IIFE is organized roughly in this order; keep new code in the matching section:

1. **DOM refs** — `document.getElementById(...)` lookups at the top.
2. **Constants** — `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (stage backdrops), `ENEMY_TYPES`, `BOSSES`.
3. **Persistence and settings** — best score, mute, difficulty, and reduced-motion flags are read from `localStorage` at startup and written back by their setters.
4. **Audio** — lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` helpers. Sound is generated procedurally; there are no audio files.
5. **Feedback effects** — hearts/bombs rendering, hit flash, combo pop, stage banner, screen shake.
6. **Input** — keyboard (`keys` map), mouse/pointer canvas mapping, and touch joystick plus FIRE/BOMB buttons.
7. **Game state** — `resetGame()`, then the spawn helpers (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), scoring (`addScore`, `breakChain`), and weapons (`fireWeapon`, `applyPowerUp`, `useBomb`).
8. **Boss patterns** — `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `updateBoss` picks and runs them.
9. **Main loop** — `update(dt)` for logic, `draw()` for rendering, and `loop(now)` driving `requestAnimationFrame`. `dt` is clamped to 50 ms.
10. **Game flow** — `loseLife`, `endGame`, `togglePause`, and the start button handler.

## Conventions

- Plain ES2015+ JavaScript and Canvas 2D. No frameworks, no bundler, no external dependencies. Do not add a CDN or npm dependency without a clear reason.
- Keep everything in `index.html` unless the user asks for a split. Scoped inside the IIFE, so avoid leaking globals.
- Use the existing `const`/`let` style, 2-space indentation, single quotes, and short arrow callbacks, matching the surrounding code.
- Constants are UPPER_SNAKE_CASE objects/arrays with the data tables at the top of the script. Balance changes (speeds, HP, spawn rates, difficulty values) belong in those tables, not inline in logic.
- Entities are plain objects with `x`, `y`, `w`, `h`, and update through the lists managed in `update`. Collision uses `rectsOverlap`.
- Persisted keys are namespaced `skyfighter.*`. Keep that prefix for any new `localStorage` key, and wrap storage access so a blocked storage environment does not break the game.
- Respect reduced motion: effects gated by `reducedMotion` (screen shake, flashes, pulses) should stay gated when you add new ones.
- Announce important state changes through `announce()` so screen readers get them via the `#srLive` region.

## Accessibility and input rules

- Every control added to the HUD or overlay needs a `title` or visible label, and keyboard-reachable buttons.
- Keep the keyboard bindings documented in `#controlsHint` in sync with the actual handlers (arrows/WASD move, Space fire, X/Shift bomb, P/Esc pause).
- Touch handlers call `preventDefault` where needed to stop scrolling; keep the `{ passive: false }` option on those listeners.

## Working agreement

- Make focused edits to `index.html`; avoid reformatting unrelated sections, since the file is large and a single diff is easy to review.
- Commit with clear messages describing the gameplay or UI change.
