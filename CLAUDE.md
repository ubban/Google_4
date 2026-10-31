# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (~1,670 lines): inline CSS in `<style>`, markup for the HUD and overlays, and one inline `<script>` IIFE that holds all game logic. There is no build step, no bundler, no package manager, and no test suite.

Other files:
- `README.md` — minimal placeholder (title only).

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` and visit `http://localhost:8000`).
- **Tests:** none exist. Verify changes by playing the game in a browser (desktop and, if touching input or layout, a mobile-sized viewport).
- **Lint/build:** none configured. Keep the inline script valid JavaScript; a quick syntax check is `node --check` on the extracted script if you want one.

## Code layout inside `index.html`

- **Constants (top of script, ~L463–516):** storage keys (`skyfighter.*`), `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`. Tune gameplay here first.
- **Persistence:** `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. All reads/writes go through the helpers near the top of the script; keep new persisted settings in the same pattern.
- **Audio:** Web Audio API, created lazily via `ensureAudio()` (requires a user gesture). SFX helpers: `beep`, `noiseBurst`, and the `sfx` object.
- **Setup and input:** `resize()` scales a fixed `BASE_W × BASE_H` (480×720) canvas to fit the window (max scale 1.6). Keyboard, mouse, and touch handlers are registered near lines 760–860; touch uses a virtual joystick plus FIRE/BOMB buttons.
- **Game state:** `resetGame()` initializes state per run; `update(dt)` advances simulation; `draw()` renders each frame; `loop(now)` drives both with a clamped `dt` (max 0.05 s) and skips update/draw unless `running && !paused`.
- **Adaptive quality:** `trackPerf(dt)` watches a rolling 45-frame average and steps a `quality` level down/up when frame time is too high/low. Check it when adding expensive effects.
- **Weapons and power-ups:** `fireWeapon()` (Vulcan/Laser, 3 levels each), `applyPowerUp()`, `maybeSpawnPowerUp()`.
- **Bombs:** `useBomb()` clears the screen and damages bosses; bomb count depends on difficulty.
- **Bosses:** `spawnBoss()`, `updateBoss()`, `killBoss()`, and the `BOSS_PATTERNS` array of bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring). A boss appears after `KILLS_PER_STAGE` kills; each stage advances the theme and boss.
- **Scoring:** `addScore()` applies a kill-chain multiplier; `breakChain()` resets it when the player is hit.
- **Lives:** `loseLife()` and `endGame()`; hearts and bombs are rendered by `renderHearts()` / `renderBombs()`.
- **Accessibility:** a reduced-motion toggle (`motionBtn`) disables screen shake, hit flash, and pulse effects. `announce()` writes to the `#srLive` aria-live region. Keep both working when adding visual effects.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split. Do not introduce a build system or external dependencies without asking.
- The script runs in strict mode inside an IIFE; do not leak globals.
- Use `const`/`let`, arrow functions, and the existing naming style (camelCase functions, UPPER_SNAKE constants).
- Game-tuning numbers live in the constants block, not scattered through logic.
- Preserve the mobile layout: `touch-action: none`, safe-area insets on the HUD, and the overlay/HUD flex layout.
- Don't add new HTML `id`s without checking the script's `getElementById` references still match.

## Git workflow

- Develop on the branch designated for the session (currently `claude/gifted-tesla-hw3op1`). Do not push to `main` directly.
- Write descriptive commit messages that explain the gameplay or code change.
- Only open a pull request when explicitly asked.
