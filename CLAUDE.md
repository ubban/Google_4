# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (in a `<style>` block), and JavaScript (in a `<script>` block). There is no build step, bundler, or package manager.
- `README.md` — minimal project readme.

There are no tests, linters, or dependencies to install.

## Running the game

Open `index.html` directly in a browser. No server is required. Chromium is the easiest target for manual testing.

## Code structure (`index.html`)

All game logic lives in one IIFE inside the `<script>` tag. Within it, roughly in order:

- **DOM references** — `getElementById` lookups for the canvas, HUD, overlay, and touch controls.
- **Storage keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Persistence is `localStorage` only.
- **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0–2) up or down.
- **Config tables** — `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS` (per-boss bullet patterns). Tune gameplay here before touching logic.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is created lazily on first user gesture.
- **Input** — keyboard listeners, touch joystick and fire/bomb buttons, and mouse/canvas pointer handling.
- **Game state** — `resetGame`, `update(dt)`, `draw()`, and the `loop(now)` driver via `requestAnimationFrame`.
- **Boss and weapons** — `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `killBoss`.

## Controls

- Move: arrow keys or WASD
- Fire: Space (hold), or the fire button on touch devices
- Bomb: X or Shift
- Pause: P or Escape

Touch devices get an on-screen joystick plus fire and bomb buttons (`touchControls` is enabled when a touch device is detected).

## Gameplay facts worth knowing

- Stages advance after `KILLS_PER_STAGE` kills (currently 18). Each stage ends in a boss, chosen as `BOSSES[(stage - 1) % BOSSES.length]`.
- Difficulty (`easy` / `normal` / `hard`) controls spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- Weapons: `vulcan` and `laser`, each with 3 power levels. Collecting a weapon power-up raises the level; past level 3 it toggles the weapon type and resets to level 1.
- Power-up types: `life`, `bomb`, `weapon`.

## Conventions

- Keep the game in a single self-contained `index.html` unless the user asks for a restructure. Don't introduce a build system or external dependencies without a clear reason.
- Match the existing style: two-space indentation, `const`/`let`, arrow functions for callbacks, short section comments using `// ---------- Name ----------`.
- Keep `localStorage` access behind the existing `STORAGE_*` constants so keys stay consistent.
- Preserve accessibility hooks: the `srLive` live region for announcements and the `reduced-motion` body class.
- Preserve mobile behavior: the viewport meta tag and touch controls are part of the design.

## Verifying changes

Since there is no test suite, verify changes by loading `index.html` in a browser and playing through:

1. Start, move, and fire.
2. Collect a power-up and confirm the HUD updates.
3. Use a bomb.
4. Reach a boss and confirm the boss bar and pattern behavior.
5. Lose all lives and confirm the game-over overlay and best-score persistence.
6. Check pause, mute, difficulty, and reduced-motion toggles persist across reloads.

At minimum, check that the inline script parses in a browser console with no errors.
