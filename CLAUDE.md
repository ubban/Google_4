# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (in a `<style>` block), and JavaScript (one inline `<script>` IIFE). About 1,670 lines. |
| `README.md` | Minimal project readme. |

There is no build step, no package manager (`package.json` does not exist), no bundler, no framework, and no test suite. Do not add one unless the user asks.

## Running it

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000`).
- Nothing needs to be installed.

## Code layout inside `index.html`

The inline script is wrapped in a single `(function () { 'use strict'; ... })();` block. Its sections, in rough order:

1. **DOM references**: `document.getElementById(...)` lookups at the top of the IIFE.
2. **Persistence keys**: `STORAGE_*` constants under the `skyfighter.*` namespace (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
3. **Accessibility helper**: `announce()` writes to the `#srLive` ARIA live region.
4. **Adaptive performance**: `trackPerf()` lowers or raises a `quality` level (0–2) based on rolling frame times.
5. **Game data tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
6. **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` helpers, built on the Web Audio API.
7. **HUD and effects**: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
8. **Input**: keyboard (`keys`), touch joystick and fire/bomb buttons, and mouse aiming on the canvas.
9. **Game state and flow**: `resetGame`, `update(dt)`, `applyPowerUp`, `useBomb`, `fireWeapon`, `loseLife`, `endGame`, `togglePause`.
10. **Spawning and enemies**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`.
11. **Boss attacks**: the `fire*` pattern helpers and the `BOSS_PATTERNS` array, driven by `updateBoss`.
12. **Rendering**: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
13. **Main loop**: `loop(now)` calls `trackPerf`, then `update` and `draw` when running and not paused. `dt` is clamped to 0.05 s.

When changing game balance, prefer editing the data tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `BOSS_PATTERNS`) over touching the logic that reads them.

## Conventions

- **Keep it a single file.** Styles and scripts stay inline in `index.html` unless the user asks for a split. Do not introduce external dependencies or CDN links without being asked.
- **Vanilla JavaScript only**, ES2015+ syntax as already used (`const`/`let`, arrow functions, template literals). Match the existing style: two-space indentation, single quotes in JS, section comments of the form `// ---------- Name ----------`.
- **Canvas coordinates** use the `BASE_W = 480`, `BASE_H = 720` logical size. `resize()` scales the canvas to fit the window. Game logic should work in base coordinates, not CSS pixels.
- **Time-based movement.** Update functions take `dt` in seconds. Do not tie movement to frame count.
- **Persistence** goes through `localStorage` with the existing `skyfighter.*` keys. Add new keys under the same prefix.
- **Accessibility**: respect `prefers-reduced-motion` (already wired to the `reducedMotion` flag and a `reduced-motion` body class). Announce important state changes with `announce()` rather than only visually.
- **Touch and desktop** must both work. Any new control needs a keyboard or mouse path and a touch path.

## Known gotchas

- The `localStorage` reads and writes at startup and in the mute, motion, difficulty, and best-score handlers are not wrapped in `try/catch`. In browsers with storage blocked (some private modes or embedded frames), these calls can throw. If you touch that code, consider guarding it.
- The `quality` setting reduces effects under load. If you add expensive per-frame drawing, check that it honors `quality` and `reducedMotion`.
- Audio is created lazily. Call `ensureAudio()` from a user gesture (click or touch) before playing sounds, because browsers block audio contexts until then.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Load `index.html` in a browser and start a game.
2. Exercise the code you changed: shooting, bombs, power-ups, each enemy type, boss fights, pause, mute, difficulty switching, and game over.
3. Check both keyboard and touch controls if input code changed. Use browser dev tools to emulate a phone-sized viewport.
4. Check the browser console for errors.

If you add a test or lint setup, document the command here.

## Git workflow

- Default branch: `main`.
- Development branch for this session: `claude/gifted-tesla-ke7zkg`. Commit and push there.
- Do not force-push. Do not create pull requests unless the user asks for one.
- Keep commits focused. Write messages in the imperative mood, for example `Add boss pattern for ...` or `Fix bomb cooldown display`.

## Keeping this file current

Update `CLAUDE.md` when you change something a future session would need to know: a new file, a new build or test command, a renamed section of the game script, a new persistence key, or a new convention. Keep it short and factual.
