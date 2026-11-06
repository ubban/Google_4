# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file browser game: a Raiden-style vertical shoot-'em-up with stages, bosses, weapon upgrades, and smart bombs.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`). About 1,670 lines.
- `README.md` — minimal placeholder; no setup instructions.

There is no build system, package manager, bundler, test suite, or linter. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No build step is required.

## Code layout inside `index.html`

- **Markup**: canvas (`#gameCanvas`), overlay/menu, HUD elements (score, best, multiplier, weapon, hearts, bombs), touch controls (`#touchControls`, joystick, fire and bomb buttons), and sound/motion/pause buttons.
- **Styles**: one `<style>` block in `<head>`.
- **Script**: one IIFE (`(function () { 'use strict'; ... })();`). Everything is private to that closure; there are no globals or modules.

Within the script, the code is grouped roughly as follows:

| Section | Purpose |
| --- | --- |
| Element lookups and `STORAGE_*` keys | DOM references and `localStorage` keys prefixed `skyfighter.` |
| Adaptive performance (`trackPerf`) | Steps a `quality` level (0–2) up or down based on average frame time |
| Config tables | `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers (Web Audio, synthesized, no asset files) |
| UI feedback | Hearts/bombs rendering, hit/bomb flashes, combo pop, stage banner, screen shake |
| Settings | Mute, difficulty, and reduced-motion toggles, each persisted to `localStorage` |
| Input | Keyboard (`keys` map), touch joystick, fire/bomb buttons, and mouse on the canvas |
| Game state | `resetGame`, `update(dt)`, `draw()`, `spawnEnemy`, `spawnBoss`, `applyPowerUp`, `useBomb`, `fireWeapon` |
| Boss patterns | `BOSS_PATTERNS` and the `fire*` helpers (radial, spread, spiral, cross, homing, double ring) |
| Game flow | `loseLife`, `killBoss`, `endGame`, `togglePause` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw` |
| Main loop | `loop(now)` with `dt` clamped to 0.05 s; started via `requestAnimationFrame` |

Fixed logical resolution is **480×720** (`BASE_W`/`BASE_H`). The canvas is scaled to fit the window by `resize()`, capped at 1.6×.

## Conventions

- **Keep it a single file.** New game code belongs in the existing `<script>` IIFE and `<style>` block unless the user asks for a split. Do not introduce a bundler or external build step.
- **No external dependencies.** The page uses no CDN scripts or fonts. Keep it that way unless the user asks otherwise. The favicon is an inline SVG data URI.
- **Use the existing config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) for tuning numbers rather than hard-coding them in logic.
- **Time-based updates.** Movement and timers use `dt` in seconds passed to `update(dt)`. Do not tie gameplay speed to frame count.
- **Persisted settings** use the `skyfighter.*` key prefix. Wrap `localStorage` access so the game still runs if storage is unavailable, as existing code does with `Number(... ) || 0` fallbacks.
- **Accessibility hooks**: announce important state changes through `announce()` (the `srLive` live region), and honor `prefers-reduced-motion` through the `reducedMotion` setting. Keep these working when adding new effects. Screen shake and flashes should respect reduced motion.
- **Touch and keyboard parity.** Any new player action needs a keyboard binding and, where relevant, a touch control.
- **Pause handling.** Update logic runs only when `running && !paused`. New timed behavior should respect the same gate.

## Testing and verification

No automated tests exist. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and play through start, a stage clear, a boss fight, and game over.
2. Check each difficulty level, mute and reduced-motion toggles, and pause/resume.
3. On a narrow or touch viewport, confirm the joystick and fire/bomb buttons work and the layout stays inside the screen.
4. Check the browser console for errors.

If you add a test harness, document how to run it here.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-1tid2h`). Do not push to other branches without permission.
- Write clear, descriptive commit messages.
- Do not open pull requests unless explicitly asked.
