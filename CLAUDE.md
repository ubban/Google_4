# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small static web project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file.
- `README.md` — minimal project readme.

There is no build step, package manager, test suite, or server-side code. The game runs by opening `index.html` in a browser (or serving the directory with any static file server).

## Running locally

```bash
# Option 1: open directly
open index.html            # macOS; use xdg-open on Linux

# Option 2: serve statically
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code layout inside `index.html`

- **`<head>`** — viewport meta (locks zoom for mobile), theme color, and an inline SVG favicon.
- **`<style>`** (≈ lines 10–359) — all styling. The HUD uses pill-style panels with heart lives; the start/pause/game-over overlay is a glassmorphic card. Reduced-motion is handled by a `body.reduced-motion` class.
- **Markup** (≈ lines 360–421) — the `#gameCanvas` element, HUD elements, overlay, touch controls, and difficulty buttons. Element IDs are referenced by name from the script.
- **`<script>`** (≈ lines 422–1667) — one IIFE with `'use strict'`. Sections, roughly in order:
  - DOM lookups and `STORAGE_*` keys
  - Adaptive performance (`trackPerf`, `quality` 0–2) that lowers effects on slow frames
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (stage backdrops), `ENEMY_TYPES`, `BOSSES`
  - Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`)
  - HUD and effects helpers (hearts, bombs, combo pops, stage banner, screen shake)
  - Input: keyboard, touch joystick, fire/bomb buttons, mouse on canvas
  - Game state: `resetGame`, `update(dt)`, `draw()`, `drawPlane`, `drawBossShip`
  - Weapons and bombs: `fireWeapon`, `useBomb`, `applyPowerUp`
  - Bosses: `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring) and `updateBoss` / `killBoss`
  - Life and round flow: `loseLife`, `endGame`, `togglePause`
  - Main loop: `loop(now)` calls `update` and `draw` via `requestAnimationFrame`, with `dt` clamped to 50 ms

## Gameplay facts worth knowing before changing code

- **Controls:** mouse move steers and click fires; arrows/WASD move; Space fires; X or Shift drops a bomb; P or Esc pauses. Touch uses a left-side drag joystick, a FIRE button, and a BOMB button.
- **Weapons:** `vulcan` and `laser`, each with power levels 1–3. Power levels add bullet spread.
- **Stages:** every `KILLS_PER_STAGE` (18) kills triggers a boss fight. Bosses cycle through `BOSSES` and their HP scales with the difficulty's `bossHpMul`.
- **Persistence (localStorage):** `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Reduced motion defaults to the `prefers-reduced-motion` media query until the user toggles it.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a bundler, framework, or npm dependencies without asking.
- **Vanilla JS, ES2015+ style.** Use `const`/`let`, arrow functions, and template literals, matching the existing code. Keep everything inside the existing IIFE; do not add new globals.
- **Match surrounding style.** Section headers use `// ---------- Name ----------`. Tuning values live in the constant tables near the top of the script rather than being hard-coded in logic.
- **Canvas coordinates.** Game logic works in `BASE_W` × `BASE_H` (480 × 720) space; `resize()` handles scaling to the viewport.
- **Accessibility.** Keep `announce()` calls for state changes that screen readers should hear, and keep the reduced-motion path working when adding visual effects (screen shake, flashes, particles).
- **Performance.** The main loop runs every frame. Avoid per-frame allocations in hot paths (`update`, `draw`, bullet/enemy loops), and gate expensive effects on `quality` where practical.

## Verification

There are no automated tests or linters configured. To verify a change:

1. Load `index.html` in a browser and check the console for errors.
2. Play through at least one stage, a boss fight, a pause/resume, and a game over.
3. Check the touch layout at phone width (or a mobile emulator), since touch controls are only shown on touch devices.
4. Confirm that best score, mute, difficulty, and reduced-motion settings persist across reloads.

If you add a non-trivial behavior or change tuning values, describe the effect in the commit message.

## Git workflow

- Develop on the branch named in the task. Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood, following the style of existing commits.
- Do not open a pull request unless asked.
