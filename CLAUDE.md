# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up played in the browser. The entire game is a single self-contained file, `index.html` (~1,670 lines): inline `<style>`, the DOM markup, and one IIFE-wrapped `<script>` that drives an HTML5 `<canvas>`.

- `README.md` – minimal project title only.
- `index.html` – the complete game. This is the only source file.

There is no build step, bundler, package manager (`package.json`), test suite, linter, or CI configuration. Do not add these unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

No install or compile step is needed.

## Code layout inside `index.html`

1. **`<style>`** (≈ lines 10–360): layout, HUD pills, glassmorphic overlay card, touch controls, animations, and the `.reduced-motion` variant. Mobile controls rely on `touch-action: none` and `pointer-events` toggles.
2. **DOM** (≈ lines 361–421): `#container` holds the canvas, vignette/flash overlays, HUD (`#hud`, `#topBar`), boss bar, combo/stage banners, a screen-reader live region (`#srLive`), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
3. **Script** (≈ lines 422–1668): one `(function () { 'use strict'; … })();` closure. Key sections, in source order:
   - DOM element references and `STORAGE_*` keys (`skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion`).
   - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
   - Audio: `ensureAudio`, `beep`, `noiseBurst` (Web Audio API; created lazily on first user gesture).
   - UI helpers: `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`, `announce`.
   - Input: window `keydown`/`keyup` (tracked in a `keys` map by `e.code`), pointer events, and virtual joystick handlers (`joystickStart/Move/End`, `canvasPointFromEvent`).
   - Simulation: `resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `update(dt)`, `applyPowerUp`, boss attack patterns (`BOSS_PATTERNS` and `fire*` helpers), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
   - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
   - Main loop: `loop(now)` clamps `dt` to 0.05 s, then calls `update` and `draw` while running and not paused. `startBtn` starts a run.

## Game design invariants

- **Canvas size:** logical resolution is `BASE_W × BASE_H = 480 × 720`; `resize()` scales it to the viewport. Game coordinates should be expressed in these base units.
- **Timing:** all movement and timers use `dt` (seconds) from `loop`, so behavior is frame-rate independent. Don't introduce per-frame increments.
- **Difficulty:** `easy`, `normal`, and `hard` each define spawn rates, enemy speed and fire chance, starting `lives`, starting `bombs`, and `bossHpMul`. Keep the three entries in sync when adding a field.
- **Progression:** a stage advances after `KILLS_PER_STAGE` (18) kills; each stage cycles through `THEMES` and triggers a boss from `BOSSES`. Tanks spawn from stage 2 (`minStage`).
- **Scoring:** the kill-chain combo multiplier (`addScore`) decays on a timer (`chainTimer`); `breakChain()` resets it when the timer expires or the player loses a life.
- **Persistence:** only `localStorage` is used, with the `skyfighter.*` keys above. Wrap new storage access so the game still works when storage is unavailable or blocked.
- **Accessibility:** respect `prefers-reduced-motion` (the `reducedMotion` flag, `.reduced-motion` class, and screen-shake/flash suppression). Announce important state changes through `announce()` / `#srLive`.
- **Audio:** audio must start only from a user gesture (`ensureAudio`). Mute state is persisted and honored by `beep`/`noiseBurst`.
- **Performance:** `trackPerf(dt)` averages the last 45 frame times and steps an adaptive `quality` level (0–2) down when the frame rate drops. Respect that mechanism when adding expensive effects, and avoid allocating large objects or arrays per frame in `update`/`draw`.
- **Mobile:** the game must remain playable with touch (joystick + fire/bomb buttons). Keyboard and pointer controls must keep working alongside it.

## Making changes

- Keep the game a single self-contained HTML file unless the user explicitly asks to split it. Do not add external dependencies or CDN scripts without asking.
- Preserve the IIFE and `'use strict'`. Don't leak globals.
- Prefer small, targeted edits. Reuse existing helpers (`beep`, `spawnExplosion`, `screenShake`, `showComboPop`, etc.) rather than duplicating logic.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for callbacks, and inline comments only where logic is non-obvious.
- When adding a new enemy, boss, power-up, or theme, extend the matching table (`ENEMY_TYPES`, `BOSSES`, `THEMES`, etc.) and its draw/update path together.

## Verification

Because there is no test harness, verify changes by:

1. Opening `index.html` in a browser (or a static server) and checking the console for errors.
2. Playing through the affected path: start a run, change difficulty, trigger the feature, pause/resume, and let it end.
3. Checking mobile layout and touch controls when the change touches UI or input (e.g., a browser devtools device emulation).
4. Checking reduced-motion and mute toggles when touching visuals or audio.

Headless browser checks (Playwright/Chromium) are a good option for automated smoke tests if the user asks for them.

## Git workflow

- Default development branch for this environment: `claude/gifted-tesla-0exxt5`. Commit and push there; don't push to other branches without explicit permission.
- Do not create pull requests unless the user explicitly asks.
- Keep commit messages descriptive, in the imperative mood, with a short summary line and a body for non-trivial changes.
