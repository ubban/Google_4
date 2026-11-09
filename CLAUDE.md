# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup (`<body>`, from line 361), CSS (`<style>`, lines 10–359), and one IIFE holding all game logic in a `<script>` block (~line 422 onward).
- `README.md` — minimal placeholder.

There is no build step, no package manager, no bundler, no test suite, and no external dependencies. Do not introduce any of these without the user asking.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by loading the page and playing; for UI or gameplay changes, check desktop and a narrow/mobile viewport since the layout and touch controls are responsive.

## Architecture (inside `index.html`)

All game code lives in one IIFE. Key pieces, in rough file order:

- **DOM references** — cached at the top of the script (`canvas`, `overlay`, HUD elements, touch controls, boss bar).
- **Persistence keys** — `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. All reads/writes should tolerate storage being unavailable.
- **Tuning tables** — change balance here first:
  - `DIFFICULTIES` (easy / normal / hard): spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: per-stage sky gradient, cloud/sun colors, and stage name (indexed as `THEMES[(stage-1) % length]`).
  - `ENEMY_TYPES` (grunt, interceptor, tank): size, HP, speed, score, and the first stage each appears in.
  - `BOSSES`: name, colors, and `hpMul`. Stage N uses `BOSSES[(N-1) % length]`, so the list cycles.
- **Audio** — Web Audio API, lazily created in `ensureAudio()` on first user gesture. Helpers: `beep()` (oscillator tones) and `noiseBurst()` (filtered noise). Respect `muted`.
- **Performance scaling** — `trackPerf()` samples frame times and sets `quality` (0 full, 1 medium, 2 low) to reduce particles/effects on slow devices.
- **Input** — keyboard (`keys` map, from `e.code`), touch (`touchControls`, virtual joystick + fire/bomb buttons), and pause via `P`/`Escape`. Bombs via `X` or `Shift`. Fire via `Space`.
- **Game loop** — `update(dt)` → `draw()` driven by `loop(now)` through `requestAnimationFrame`. `update` handles player movement, firing (`fireWeapon()`), enemy and bullet logic, collisions, and stage progression; `updateBoss(dt)` handles boss phases; `useBomb()` clears the screen and damages bosses.
- **Boss attack patterns** — `fire*` helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **Rendering** — `draw()` with helpers `drawPlane`, `drawBossShip`, `drawClouds` (parallax layers).
- **Accessibility** — `announce()` writes to a live region (`srLive`). A reduced-motion toggle disables screen shake, flashes, and the low-health pulse; it defaults from `prefers-reduced-motion` on first load.

## Conventions

- Keep it a single file unless the user asks for a split. New code belongs inside the existing IIFE; avoid leaking globals.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in JS, and small named helper functions.
- Prefer adding to the tuning tables over hard-coding numbers in logic.
- Keep the game playable with touch as well as keyboard. Any new control needs a touch equivalent.
- Keep `localStorage` access wrapped so the game still works in private/blocked-storage contexts.

## Git

- Development branch for automated work: `claude/gifted-tesla-mdzs9n`. Default branch: `main`.
- Write descriptive commit messages that explain the gameplay or UI change.
- Do not open a pull request unless the user asks for one.
