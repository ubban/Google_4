# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (CSS in a `<style>` block, JavaScript in a `<script>` block, no external assets). Audio is synthesized with the WebAudio API, and the game draws everything on a 2D canvas.

Files:

- `index.html` — the whole game (~1,670 lines).
- `README.md` — minimal placeholder; not a source of truth for the game.
- `CLAUDE.md` — this file.

There is no `package.json`, build step, bundler, test suite, linter, or CI configuration. To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Running and verifying changes

- No build or install is needed. Open `index.html` directly, or run `python3 -m http.server` from the repo root and visit `http://localhost:8000`.
- There are no automated tests. Verify behavior by playing the game in a browser, including the affected code path. For changes to input, layout, or HUD, check both desktop (keyboard/mouse) and a narrow/touch viewport, since the game has touch controls and responsive sizing.
- Because the file is one big script, a syntax error breaks the whole game. After editing, at minimum load the page and confirm there are no console errors.

## Code layout inside `index.html`

The `<script>` block is one IIFE-style scope organized by `// ---------- Section ----------` comments. The main sections, in order:

1. **DOM references** — `getElementById` lookups for the canvas, HUD, overlay, and touch controls.
2. **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*` in `localStorage`).
3. **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low).
4. **Tuning data** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, and `BOSSES`. Most balance changes belong here rather than in the logic.
5. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is created lazily on the first user gesture (required by browsers).
6. **Canvas sizing** — `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the window (capped at 1.6x).
7. **Input** — keyboard (`keys`), touch joystick and fire/bomb buttons, and mouse steering that glides the plane toward the pointer.
8. **Game state** — `resetGame`, `score`, `lives`, `running`, `paused`, `gameOver`, and related globals.
9. **Drawing helpers** — `drawPlane`, `drawBossShip`, `drawClouds`, and the main `draw()`.
10. **Spawning and combat** — `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `useBomb`.
11. **Update** — `update(dt)` (player, enemies, bullets, collisions, stage progression) and `updateBoss(dt)`.
12. **Game flow** — `endGame()` (saves best score, shows overlay) and `loop(now)` (requestAnimationFrame driver with `dt` capped at 0.05 s).

Key behavior to keep in mind:

- The loop calls `update(dt)` and `draw()` only when `running && !paused`.
- Difficulty (`easy` / `normal` / `hard`) controls spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- Each stage requires `KILLS_PER_STAGE` kills; then a boss spawns (with a warning banner). Defeating the boss advances `stage`, and the sky theme cycles through `THEMES` by `(stage - 1) % THEMES.length`.
- Weapons: `vulcan` (spread grows with level: 1, 2, or 3 shots) and `laser`. Weapon pickups raise the level; past level 3 they toggle between `vulcan` and `laser` and reset the level to 1.
- Best score is persisted to `localStorage` only when beaten.

## Conventions

- Keep everything in `index.html` unless the user asks for a structural change. Do not introduce a framework, bundler, or external CDN dependency without an explicit request — the single-file, no-network design is intentional.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, and the `// ---------- Section ----------` banners.
- Gameplay numbers live in the tuning tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer editing those over hard-coding new magic numbers in logic.
- Anything that adds a new `localStorage` key should follow the `skyfighter.*` naming pattern and be declared alongside the existing `STORAGE_*` constants. Wrap storage access so the game still works if storage is unavailable (private mode, blocked site data).
- Accessibility features are deliberate: `prefers-reduced-motion` support (`reducedMotion`), a screen-reader live region (`srLive` / `announce()`), and a mute toggle. Preserve them when changing related UI.
- Keep the UI usable on phones: the page uses `touch-action: none`, `viewport-fit=cover`, and safe-area insets. Don't add scrolling or fixed-width layouts.

## Git and workflow

- Default branch work happens on the branch the session specifies; do not push to other branches without permission.
- Keep commits focused and describe the gameplay or UI change in the message.
- Do not create pull requests unless explicitly asked.

## Updating this file

Update `CLAUDE.md` when you change the project's structure (new files, a split of `index.html`, a build step, tests, or CI), add or rename major sections of the game script, or change the conventions above. Keep it factual and short; avoid restating what is obvious from reading the code.
