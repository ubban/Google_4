# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** is a single-file, browser-based, Raiden-style vertical shoot-'em-up. There is no build step, no package manager, and no test suite. The whole game lives in `index.html`:

- `<style>` (approx. lines 10–360): all CSS, including HUD, overlays, touch controls, and dark color-scheme styling.
- Markup (approx. lines 363–421): `#gameCanvas`, `#hud`, `#touchControls`, `#overlay`, and related elements.
- `<script>` (starts at line 422): one IIFE containing all game logic, rendering, audio, and input.

`README.md` is a placeholder and contains no project documentation.

## Running and checking changes

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, including the affected code path (pause, bombs, boss fights, difficulty changes, mute, reduced motion, and touch controls where relevant).
- Keep the game self-contained. Do not add external scripts, stylesheets, or fonts unless the user asks for them.

## Code layout inside the script

The script is organized roughly in this order. Keep new code in the matching section.

| Area | Notes |
| --- | --- |
| Constants | `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Persistence | `bestScore`, `muted`, `difficulty`, and `reducedMotion` are read from and written to `localStorage` |
| Audio | Web Audio API via `ensureAudio()`, with the `beep`, `noiseBurst`, and `sfx` helpers |
| Canvas sizing | Logical resolution is `BASE_W = 480` × `BASE_H = 720`, scaled to fit the viewport in `resize()` |
| Input | Keyboard handlers (`keydown`/`keyup`) and touch joystick and buttons |
| Entities | `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `fireWeapon`, and the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireHomingSeeded`, and others) |
| Simulation | `update(dt)` and `updateBoss(dt)` |
| Rendering | `draw()`, `drawPlane`, `drawBossShip`, `drawClouds` |
| Main loop | `loop(now)`, which drives update and draw each frame |

## Game rules to keep in mind

- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier. Changing a value here changes the balance of the whole game.
- **Stages** advance every `KILLS_PER_STAGE` kills. Each stage uses the next theme in `THEMES` and the next boss in `BOSSES`.
- **Enemies** are `grunt`, `interceptor` (weaving, fast), and `tank` (high HP, introduced at stage 2). `minStage` gates when each type appears.
- **Weapons** are `vulcan` (default) and `laser`, toggled by a pickup at `update` around line 1315. `player.level` controls spread width.
- **Controls:** Arrow keys or WASD move. Space fires. X, Shift, or the bomb button triggers a smart bomb. P or Escape pauses. Touch devices get a joystick, fire button, and bomb button.

## Conventions

- Keep the file as a single self-contained HTML file unless the user asks for a split.
- Preserve the existing style: 2-space indentation, single quotes in JavaScript, and short section comments with `// ---------- Name ----------` headers.
- Wrap all `localStorage` reads and writes so the game still works when storage is unavailable (private windows, blocked site data).
- Respect `reducedMotion` for any new screen shake, flashes, or particle-heavy effects. Announce important state changes through the existing `announce()` / `srLive` live region.
- Keep the layout usable at phone width. The page uses `touch-action: none` and `overflow: hidden`, so do not introduce scrolling content.
- Do not commit generated or temporary files.

## Git

- Development branch for this session: `claude/gifted-tesla-46swk3`. Do not push to other branches without permission.
- Do not create pull requests unless the user explicitly asks.
