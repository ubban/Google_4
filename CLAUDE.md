# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (one inline `<script>` wrapped in an IIFE, `'use strict'`).
- `README.md` — a one-line title. Keep it accurate if the project description changes.

There is no `package.json`, build step, bundler, test suite, or linter. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify changes by playing them in a real browser. Syntax-only checks are not enough for gameplay, rendering, or audio changes.
- Test both desktop (keyboard) and touch layouts when touching input, HUD, or layout code. The `#touchControls` joystick and fire/bomb buttons appear on touch devices.

## Architecture of `index.html`

Everything lives in one file, so keep sections organized and avoid duplicating helpers.

- **Markup/CSS (top of file):** HUD pills (`#hud`, `#hearts`, `#bombs`, `#weaponStat`, `#multStat`), overlay card (`#overlay`), boss bar (`#bossBarWrap`), vignette/flash effects, and touch controls. Dark color scheme (`color-scheme: dark`).
- **Config tables (in the script):**
  - `DIFFICULTIES` — `easy` / `normal` / `hard`: spawn timing, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — sky/cloud/sun palettes, cycled per stage.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
  - `BOSSES` — six bosses (VANGUARD → TITAN) with colors and HP multipliers.
  - `BOSS_PATTERNS` — bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring).
- **Persistence:** `localStorage` keys prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Always wrap storage access in a way that tolerates it being unavailable (private mode, blocked storage).
- **Game loop:** `update(dt)` advances state, `draw()` renders, `loop(now)` drives both via `requestAnimationFrame`. `trackPerf` adapts a `quality` level (0–2) to frame time; respect it when adding expensive effects.
- **Audio:** Web Audio via `ensureAudio()`, with `beep()` and `noiseBurst()` helpers. Audio must start from a user gesture.
- **Accessibility:** `announce()` writes to the `#srLive` live region. `reducedMotion` (from preference or the user toggle) gates screen shake and similar motion effects.
- **Weapons:** `player.weapon` is `'vulcan'` or `'laser'`, each with a power level. Weapon changes come from power-ups (`applyPowerUp`).
- **Bombs:** `useBomb()` clears the screen and damages bosses. Triggered by `X` or Shift on keyboard, or the bomb button on touch.

## Controls (for testing)

- Move: arrow keys or `WASD`; mouse or touch also moves the ship.
- Fire: `Space` (hold), or hold the fire button on touch.
- Bomb: `X` or `Shift`.
- Pause: `P` or `Esc`.

## Conventions

- Keep the project as a single self-contained HTML file unless the user asks for a restructure. Do not introduce frameworks, CDNs, or build tooling without approval.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions in callbacks, section comments with `// ---------- Name ----------`.
- Keep game balance in the config tables rather than hard-coding numbers in logic.
- New visual effects should respect `reducedMotion` and the `quality` level.
- Keep the UI usable at phone widths. The layout uses `viewport-fit=cover` and `env(safe-area-inset-*)`.

## Git

- Default development branch for this work is `claude/gifted-tesla-o13cgq`. Do not push to other branches without explicit permission.
- Commit messages: a short imperative subject, then a body explaining the why when it isn't obvious.
