# CLAUDE.md

Guidance for working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas game.

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines).
- `README.md` — minimal placeholder.

There is no build system, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser. No server or install step is needed.

For a quick local check with a server:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code layout inside `index.html`

- **Markup (lines ~362–420):** `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), boss bar, combo/stage banners, top-bar buttons (mute, reduced motion, pause), touch controls (joystick, FIRE, BOMB), and the start/pause `#overlay`.
- **Script (lines ~422–1668):** one IIFE (`'use strict'`) containing all game logic. Sections are marked with `// ---------- Name ----------` comments. Keep new code inside that IIFE and follow the same section style.

Key subsystems and where they live (approximate line numbers; search by name if they drift):

| Subsystem | Identifiers |
| --- | --- |
| Persistence keys | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` |
| Adaptive quality | `quality`, `trackPerf()` — drops to medium/low effects when frame time is slow |
| Difficulty presets | `DIFFICULTIES` (easy / normal / hard), `setDifficulty()` |
| Stages and themes | `KILLS_PER_STAGE`, `THEMES` (sky palettes, cycled per stage) |
| Enemies | `ENEMY_TYPES` (grunt, interceptor, tank), `spawnEnemy()` |
| Bosses | `BOSSES` (appearance, `hpMul`), `BOSS_PATTERNS`, `spawnBoss()`, `updateBoss()`, `killBoss()` |
| Weapons | `fireWeapon()`, `player.weapon` (`vulcan` / `laser`), `player.level` 1–3 |
| Power-ups | `maybeSpawnPowerUp()`, `applyPowerUp()` — types `life`, `bomb`, `weapon` |
| Bombs | `useBomb()`, `flashBomb()` |
| Combo / score | `addScore()`, `breakChain()`, `showComboPop()` |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx` — WebAudio synth, no audio files |
| Rendering | `drawPlane()`, `drawBossShip()`, `drawClouds()`, `draw()` |
| Game loop | `update(dt)` → `draw()` inside `loop(now)`; `dt` is clamped to 50 ms |
| Input | keyboard (arrows/WASD, Space fire, X/Shift bomb, P/Esc pause), mouse (`canvasPointFromEvent`), touch (`joystickStart/Move/End`) |

### Conventions in this file

- Plain ES2015+ JavaScript, no modules, no dependencies. Do not introduce libraries or CDN scripts without a clear reason.
- Constants are `UPPER_SNAKE_CASE` objects or values; game entities are plain objects in arrays.
- Time-based motion multiplies by `dt` (seconds). Do not use frame counts for movement.
- Persisted state goes through `localStorage` with a `skyfighter.*` key. Wrap storage access so the game still runs if storage is blocked, and add a new `STORAGE_*` constant rather than an inline string.
- Accessibility hooks already exist: `announce()` feeds the `aria-live` region, and a reduced-motion mode disables screen shake, flashes, and pulses. New visual effects that shake or flash must respect `reducedMotion`.
- Keep the visual style consistent with the existing HUD/overlay CSS (pill-style stats, glassmorphic card) and the canvas drawing helpers.

## Making changes

- Make edits in `index.html` with minimal, focused diffs. Do not reformat unrelated code.
- When adding a new enemy, boss, power-up, or difficulty setting, extend the existing config table (`ENEMY_TYPES`, `BOSSES`, `DIFFICULTIES`) and the matching spawn/apply logic, rather than adding special cases elsewhere.
- Changing the stage progression (`KILLS_PER_STAGE`, `THEMES`) or difficulty numbers changes game balance. Mention it in the commit message.

## Verifying changes

There is no automated test suite. Verify by:

1. Opening `index.html` in a browser (or the local server above) with no console errors.
2. Playing through the affected path: start a game, exercise the changed mechanic, pause/resume, use a bomb, and lose a life to game over.
3. Checking both desktop (keyboard/mouse) and touch layouts if HUD, overlay, or input code changed. Resize the window to confirm the canvas still scales (`resize()`).
4. Toggling mute and reduced motion if audio or visual effects changed.

If you cannot run a browser, say so explicitly in your summary and describe what was verified by reading the code.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-91y3s1`). Do not push to `main` directly.
- Write descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless explicitly asked.
