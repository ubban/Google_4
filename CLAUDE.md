# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in an HTML5 canvas.

- `index.html` — the entire game: markup, CSS, and JavaScript in one self-contained file (~1,670 lines).
- `README.md` — a placeholder; it has no setup or usage instructions.

There is no build step, no package manager, no bundler, and no test suite. Do not introduce any of these unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000
```

Verify changes by playing them in a browser. There are no automated tests or linters to run.

## Code layout of `index.html`

- **`<style>`** (about lines 10–360): all CSS. The HUD uses pill-style panels, and the overlay is a glassmorphic card. Mobile layout is handled here too.
- **`<body>`** (about lines 361–421): the canvas (`#gameCanvas`), HUD, overlays, and touch controls.
- **`<script>`** (about line 422 onward): one IIFE `(function () { ... })()` that holds all game logic. Nothing is exposed on `window`; keep it that way.

Within the script, the code is grouped roughly in this order:

1. **Persistence keys and tuning tables**: `STORAGE_*` keys, `DIFFICULTIES`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
2. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is created lazily on the first user gesture.
3. **HUD and UI helpers**: hearts, bombs, combo pop-ups, stage banners, screen shake, mute/difficulty/motion toggles.
4. **Input**: keyboard handlers, a virtual joystick and fire/bomb buttons for touch, and mouse steering on the canvas.
5. **Game state and logic**: `resetGame`, `update(dt)`, spawning (`spawnEnemy`, `spawnBoss`, power-ups), weapons (`fireWeapon`), bombs (`useBomb`), boss attack patterns (`BOSS_PATTERNS`), and the kill-chain score multiplier.
6. **Rendering**: `draw()` with helpers `drawPlane`, `drawBossShip`, `drawClouds`.
7. **Main loop**: `loop(now)` runs on `requestAnimationFrame`. The logical resolution is fixed at `BASE_W × BASE_H` (480 × 720) and scaled by `resize()`.

## Conventions

- **Match the existing style.** The file uses 2-space indentation, single quotes, arrow functions for callbacks, `const` by default, and short inline comments only where the logic is not obvious.
- **Keep it a single file.** Put new CSS in `<style>`, new markup in `<body>`, and new code inside the existing IIFE. Do not split the game into modules or add external dependencies without an explicit request.
- **Use the tuning tables.** Balance changes (speeds, HP, spawn rates, lives, bombs) belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE`, not as magic numbers in the logic.
- **Persist settings through the `skyfighter.*` keys.** Stored values are read defensively with fallbacks, such as `Number(localStorage.getItem(...)) || 0`. Keep that pattern for any new setting.
- **Keep the game responsive.** Both desktop and touch input must keep working. Respect the reduced-motion setting when adding screen effects.

## Controls (for reference when testing)

- Move: mouse over the canvas, or Arrow keys / WASD. On touch, drag on the left side.
- Fire: Space, hold the mouse button, or the FIRE button.
- Bomb: `X` or Shift, or the BOMB button.
- Pause: `P` or Esc.

## Git

- Develop on the branch the session specifies. Do not push to `main` directly.
- Write commit messages that describe the gameplay or code change, for example "Add ..." or "Fix ...", with a short body when the reason is not obvious.
- Do not open a pull request unless one is requested.
