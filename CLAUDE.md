# CLAUDE.md

Guidance for Claude Code working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game lives in one file, `index.html`: inline CSS in `<style>`, markup for the HUD and overlays, and one inline `<script>` IIFE that holds all game logic. `README.md` is a placeholder.

There is no build step, no package manager, no bundler, and no test suite. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify changes by playing them in a real browser. Syntax checks alone will not catch gameplay regressions.
- Test at both desktop and phone-sized viewports. The canvas scales to fit, and touch controls appear on touch devices.

## Code layout (inside `index.html`)

- **Config constants** (near the top of the script): `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, and `BASE_W`/`BASE_H` (480×720 logical canvas size). Balance tuning should happen here first.
- **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all under the `skyfighter.*` prefix in `localStorage`.
- **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `EXPLODE_SFX` mapping. Audio is created lazily on the first user gesture.
- **Input**: keyboard handlers on `window` (`keys[e.code]`), pointer/touch joystick and buttons, and `canvasPointFromEvent()`.
- **Update/draw**: `update(dt)` advances simulation state, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`. `dt` is clamped to 0.05 s.
- **Combat**: `spawnEnemy`, `spawnBoss`, `fireWeapon`, `updateBoss`, the `BOSS_PATTERNS` bullet-pattern functions, `useBomb`, `loseLife`, `killBoss`, `endGame`.
- **Pause**: `togglePause()`, toggled by `P` or `Escape`.

## Controls (for manual testing)

- Move: arrow keys or WASD.
- Fire: hold `Space` (or touch fire button).
- Bomb: `X` or Shift.
- Pause: `P` or Escape.
- Weapon switching is handled around line ~1315 (vulcan ↔ laser via a power-up).

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce frameworks or external dependencies. The only external resource is the Segoe UI/Roboto/Arial font stack.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for handlers, and section comments of the form `// ---------- Name ----------`.
- Keep the game running at 60 fps on modest hardware. Avoid per-frame allocations in `update`/`draw` where practical.
- Preserve accessibility hooks: the `reducedMotion` toggle (which also respects `prefers-reduced-motion`), the `srLive` live region for announcements via `announce()`, and the `user-scalable=no` viewport setup used for the game.
- Persist only the four `skyfighter.*` keys. Wrap any new storage access so the game still works when `localStorage` is unavailable.

## Making changes

- Read the relevant section of `index.html` before editing. The file is large, so use targeted searches rather than reading it whole.
- Make small, focused edits. Do not reformat unrelated code.
- After a change, play the affected path: start a game, exercise the changed mechanic, and check the HUD and pause/game-over flow.
- Git: develop on the branch the session specifies. Do not push to other branches without explicit permission. Do not open a pull request unless asked.
