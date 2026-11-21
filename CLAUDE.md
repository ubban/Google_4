# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a Raiden-style vertical shoot-em-up played in the browser, built as a single HTML5 canvas page. The repo is intentionally minimal:

- `index.html` — the entire game: markup, CSS (`<style>`), and JavaScript (`<script>`), all inline (~1,670 lines).
- `README.md` — a placeholder title only.

There is no build step, no package manager, no bundler, no framework, and no external assets. Audio is synthesized with the WebAudio API, and all graphics are drawn procedurally on the canvas.

## Running and verifying

- **Run:** open `index.html` directly in a browser. No server is required.
- **Tests / lint / build:** none exist. Don't invent commands for them. Verify changes by loading the page in a browser and playing through the affected behavior.
- For UI or gameplay changes, check both desktop (keyboard/mouse) and touch layouts. Touch controls only appear when the device reports touch support.

## Code layout (inside `index.html`)

The game lives in one IIFE (`(function () { 'use strict'; ... })()`). Major sections, in file order, marked by `// ----------` comments:

1. **DOM refs and storage keys** — element lookups by id, and `skyfighter.*` localStorage keys.
2. **Adaptive performance** — `trackPerf` drops render quality (`quality` 0–2) when frame time is high.
3. **Config tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
4. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is created lazily on first user gesture.
5. **HUD helpers** — hearts, bombs, combo pops, stage banners, screen shake, flash.
6. **Canvas sizing** — `BASE_W = 480`, `BASE_H = 720`, scaled by `resize()`.
7. **Input** — keyboard (`keys[e.code]`), mouse steering, virtual joystick, and fire/bomb touch buttons.
8. **Game state** — `resetGame()` and the entity arrays (bullets, enemies, power-ups, particles).
9. **Entities and drawing** — `drawPlane`, `drawBossShip`, spawners, collisions, `spawnExplosion`.
10. **Weapons and bombs** — `fireWeapon` (Vulcan with 3 levels, Laser), `useBomb`.
11. **Update** — `update(dt)`, scoring via `addScore` / `breakChain`, `applyPowerUp`.
12. **Bosses** — `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing, double ring) driven by `updateBoss`; `killBoss`.
13. **Lifecycle** — `loseLife`, `endGame`, `togglePause`.
14. **Render** — `drawClouds` (parallax) and `draw()`.
15. **Main loop and start button** — `loop(now)` caps `dt` at 0.05s and runs update + draw only when `running && !paused`.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Don't add a build system or dependencies without asking.
- **No external network requests or assets.** Keep the game self-contained and working offline.
- **Frame-based timing uses `dt` in seconds.** Scale movement and timers by `dt`; don't assume a fixed frame rate.
- **Respect the pause/game-over guards.** Actions that change game state should check `running`, `paused`, and `gameOver` (see `useBomb`, `update`, and the loop).
- **Balance lives in the config tables.** Prefer tuning values in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE` over hard-coding numbers in logic.
- **Accessibility hooks exist — keep them working.** This includes the `aria-live` announcer (`announce`), the reduced-motion toggle (`body.reduced-motion`, which also respects `prefers-reduced-motion`), and keyboard operation of the buttons.
- **Persistence is localStorage only.** Keys are prefixed `skyfighter.`. Wrap new storage access so the game still works when storage is unavailable.
- **Keep the HUD visual language.** Pill-style HUD, glassmorphic overlay card, and the existing color/theme tokens in the stylesheet.

## Controls (for reference when testing)

- **Keyboard:** Arrows/WASD move, Space fire, X or Shift bomb, P or Esc pause.
- **Mouse:** move to steer, click to fire.
- **Touch:** drag left zone to fly, FIRE and BOMB buttons on the right.
- Difficulty (easy / normal / hard) is chosen on the title screen and persisted.

## Git workflow

- Develop on the branch assigned to the session; don't push to other branches without permission.
- Write commit messages in the imperative mood, with a summary line and an explanatory body for non-trivial changes.
- Don't create pull requests unless the user asks.
