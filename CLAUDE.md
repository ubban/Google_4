# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The player flies a plane up a scrolling sky, switches between two weapons, drops limited smart bombs, collects power-ups, and fights a boss at the end of each stage.

There is no build step, no package manager, and no dependencies. Everything lives in one file.

## Repository layout

| Path         | Purpose                                                                 |
|--------------|-------------------------------------------------------------------------|
| `index.html` | The whole game: CSS (inline `<style>`, lines ~10–359), markup (`<body>`, lines ~361–421), and JS (inline `<script>`, lines ~422–1667). |
| `README.md`  | Minimal project readme.                                                 |
| `CLAUDE.md`  | This file.                                                              |

## Running and testing

- **Run:** open `index.html` directly in a browser. No server is required.
- **Build:** none.
- **Tests:** none exist. There is no test runner, linter, or formatter configured.
- **Verify changes manually** in a browser. Check desktop (keyboard and mouse) and touch controls, since both input paths are implemented.

## Code architecture

All JavaScript runs inside one IIFE (`(function () { 'use strict'; ... })();`). Nothing is exported or attached to `window`. Keep new code inside that scope.

Sections inside the script are marked with comment banners such as `// ---------- Input ----------`. Follow that pattern when adding a section.

Main sections, in file order:

- **DOM references and constants:** cached `getElementById` lookups, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Adaptive performance:** `trackPerf` samples frame time and steps a `quality` level (0 full, 1 medium, 2 low) up or down.
- **Audio:** `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object. Audio is created lazily on the first user gesture, which browsers require.
- **HUD and effects:** `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`.
- **Input:** `keys` map for keyboard, joystick helpers for touch, mouse steering on the canvas, and explicit fire and bomb buttons.
- **Game state:** `resetGame()` initializes all mutable state. Per-entity spawning is in `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, and `spawnExplosion`.
- **Combat:** `fireWeapon`, `useBomb`, `addScore`, and the kill-chain multiplier (`breakChain`).
- **Simulation:** `update(dt)` runs game logic each frame. `applyPowerUp` handles pickups.
- **Boss patterns:** a set of projectile emitters (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) composed into `BOSS_PATTERNS`, driven by `updateBoss`.
- **Rendering:** `draw()` renders each frame. Helpers include `drawPlane`, `drawBossShip`, and `drawClouds` for the parallax layers.
- **Loop:** `loop(now)` clamps `dt` to 0.05 s, runs `update` and `draw` only when running and not paused, then calls `requestAnimationFrame`.

### Game rules worth knowing

- **Difficulty:** `easy`, `normal`, and `hard`. Each sets spawn timing, enemy speed, shoot chance, starting lives and bombs, and a boss HP multiplier. Changing values here changes balance for the whole game.
- **Enemies:** `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2).
- **Stages:** `KILLS_PER_STAGE` kills trigger a boss. Stages cycle through `THEMES` for sky palette and name.
- **Power-ups:** `life`, `bomb`, and `weapon`. Weapon pickups raise the level from 1 to 3, then toggle between `vulcan` and `laser`.
- **Persistence:** `localStorage` keys are `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Treat these as a public contract. Renaming one resets that player's saved setting.
- **Pause:** `P` or `Escape`. Pause is also reachable from the on-screen button.
- **Accessibility:** a `prefers-reduced-motion` media query sets the initial reduced-motion state, which the user can override. Screen-reader announcements go through `announce()` into the `srLive` live region.

## Conventions

- **Vanilla JavaScript only.** Do not add frameworks, bundlers, or npm dependencies unless the user asks. The single-file, open-in-browser setup is intentional.
- **Keep it in one file** unless the user asks for a split. If a split is requested, use plain `<script src>` tags and keep the no-build property.
- **Match the surrounding style:** two-space indentation, single quotes, `const` by default, arrow functions for callbacks, and short comment banners for sections.
- **Use the existing helpers.** Reuse `announce`, `sfx.*`, `spawnExplosion`, and `addScore` instead of duplicating their logic.
- **Mind performance.** The loop runs every frame, and `quality` exists to shed work on slow devices. Avoid per-frame allocations in hot paths, and respect the `quality` level when adding effects.
- **Preserve touch support.** Any new control needs both a keyboard or mouse path and a touch path, with `{ passive: false }` where `preventDefault` is called.

## Git

- Develop on the branch the session specifies. Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood, matching the existing history (for example, `Add Sky Fighter: ...`).
- Do not open pull requests unless the user asks for one.
