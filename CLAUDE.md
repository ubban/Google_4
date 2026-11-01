# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a single-file, browser-based vertical shoot-'em-up (Raiden-style) written in plain HTML, CSS and JavaScript with no build step, no dependencies and no package manager.

- `index.html` — the entire game: markup (~lines 1–360), inline `<style>` (~lines 10–360), and one inline `<script>` IIFE (~lines 422–1668).
- `README.md` — minimal project title only.

There is no test suite, linter, bundler or CI configuration in the repo.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

No install step is needed.

## Code layout inside `index.html`

The script is wrapped in an IIFE with `'use strict'`. Sections, roughly in file order:

| Area | What it does |
| --- | --- |
| DOM lookups | Caches all element references (`canvas`, HUD elements, overlay, touch buttons). IDs must stay in sync with the markup. |
| Storage keys | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`. |
| Adaptive quality | `trackPerf()` steps a `quality` level (0–2) based on a rolling frame-time average. |
| Config tables | `DIFFICULTIES` (easy/normal/hard), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses), `KILLS_PER_STAGE = 18`. |
| Audio | Web Audio synthesis: `ensureAudio()`, `beep()`, `noiseBurst()`, and a named `sfx` object. No audio files. |
| HUD / feedback | Hearts and bombs rendering, combo pop, stage banner, screen shake, hit/bomb flash, screen-reader announcements via `announce()`. |
| Input | Keyboard (`keys` map), mouse (steer and fire on canvas), virtual joystick and buttons for touch. |
| Canvas sizing | `resize()` keeps a fixed 480×720 logical canvas (`BASE_W`/`BASE_H`) scaled to the window (max 1.6×). |
| Game state | `resetGame()` initialises everything; `update(dt)` runs game logic; `draw()` renders. |
| Spawning | `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()` (life 6%, bomb 8%, weapon 20% of drops). |
| Weapons | `fireWeapon()` — `vulcan` (spread of 1–3 shots) and `laser`; weapon power-ups raise level 1→3, then toggle weapon type. |
| Bosses | `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring) driven by `updateBoss()`; `killBoss()`. |
| Lifecycle | `loseLife()` (1.5 s invulnerability), `endGame()`, `togglePause()`. |
| Main loop | `loop(now)` clamps `dt` to 0.05 s, runs `update` and `draw` while running and not paused, then re-queues via `requestAnimationFrame`. |

### Game rules worth knowing

- Difficulty sets spawn rate, enemy fire chance, starting lives, starting bombs and boss HP multiplier.
- Score multiplier is built by a kill chain: `1 + min(9, floor(chain / 5))`, so up to ×10. `breakChain()` resets it, and losing a life breaks it.
- Bombs are triggered by `X`, `Shift`, or the BOMB button.
- At game over, `endGame()` writes the best score to `localStorage` only if the run beat the stored value.

## Conventions

- **Keep it dependency-free.** Do not add npm packages, CDN scripts or a build step unless the user explicitly asks for one.
- **Single file.** Keep the game in `index.html` unless there is a clear reason to split it out.
- **Preserve the logical resolution.** Game coordinates assume a 480×720 canvas; scale via `resize()`, not by changing the base size.
- **Respect reduced motion.** New screen-shake, flash or pulse effects should be gated on `reducedMotion`.
- **Keep audio synthesized.** Add sounds through the existing `beep()` / `noiseBurst()` helpers rather than loading assets.
- **Mute and accessibility.** Any new user-facing message should go through `announce()` so screen readers receive it, and new buttons need a `title` or visible label.
- **Touch and keyboard parity.** Any new control must work on both touch controls and keyboard/mouse.
- **Performance.** Hot-path code runs every frame; avoid per-frame allocations where practical, and respect the `quality` level when adding expensive effects.

## Verifying changes

Since there is no automated test setup, verify by:

1. Loading `index.html` in a browser and checking the browser console for errors.
2. Playing through a start, a few stages, a boss fight, a pause/resume, and a game over.
3. Checking touch controls with browser dev-tools device emulation when touching input code.
4. Checking mute, reduced-motion and difficulty toggles persist across reloads.

If you add a test harness or tooling, document the commands here.

## Git

- Default development branch for this work: `claude/gifted-tesla-sxlar1`.
- Write clear, descriptive commit messages in the imperative mood.
