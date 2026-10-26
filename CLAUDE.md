# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-em-up. There is no build step, no package manager, no bundler, and no test suite.

Files:

- `index.html` — the whole game: inline `<style>`, markup, and one inline `<script>` (an IIFE with `'use strict'`). About 1,670 lines.
- `README.md` — a placeholder title only.

## Running it

Open `index.html` in a browser. There is nothing to install or compile. For a quick local server, `python3 -m http.server` from the repo root works, but it is not required.

## Verifying changes

There are no automated tests or linters configured. Verify changes by playing the game in a browser:

- Check the title overlay, Start, and the Pause button (`⏸`).
- Check each difficulty (Easy / Normal / Hard) and that the choice persists after reload.
- Check the weapon upgrades, bombs, and power-ups.
- Reach stage 2+ to see tank enemies and the theme change (each stage uses `THEMES[(stage - 1) % THEMES.length]`).
- Defeat a boss (each stage's boss is `BOSSES[(stage - 1) % BOSSES.length]`), and check the boss health bar.
- On a narrow or touch viewport, check the joystick and the Fire/Bomb buttons.
- Toggle mute and reduced motion, and check that both persist.

If you add a test setup, document the command here.

## Code layout (inside the main `<script>`)

Roughly in order:

1. **DOM references** — `const xxxEl = document.getElementById(...)`. Element IDs in the markup are the contract; keep them in sync.
2. **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`. Changing a key orphans players' saved values.
3. **Adaptive quality** — `trackPerf(dt)` samples the last 45 frame times and steps `quality` (0 full, 1 medium, 2 low) up or down.
4. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`. Most balance changes belong here, not in the update loop.
5. **Audio** — a lazily created Web Audio context (`ensureAudio`), a `beep` / `noiseBurst` pair, and an `sfx` object of named sounds. Audio starts only after a user gesture.
6. **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
7. **Input** — keyboard (`keys` map, `e.code`), mouse aiming on the canvas, and touch joystick plus Fire and Bomb buttons.
8. **Game state** — `resetGame()` is the single place that initializes all run state. Add new run state there.
9. **Drawing** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
10. **Simulation** — `update(dt)` drives movement, collisions, spawning, and scoring. `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `addScore` / `breakChain`, `useBomb`, `fireWeapon`, `applyPowerUp`.
11. **Boss patterns** — `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, and others). `updateBoss` picks and runs them.
12. **Lifecycle** — `loseLife`, `endGame`, `togglePause`, and `killBoss`.
13. **Loop** — `loop(now)` caps `dt` at 0.05 s, then calls `update` and `draw` while the game is running and not paused. Start is handled by `startBtn`.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not add a build step or a framework without a request.
- Match the existing style: `const` / `let`, arrow functions for callbacks, two-space indentation, single quotes, and short section comments of the form `// ---------- Name ----------`.
- Game logic uses `dt` in seconds. Speeds are in canvas pixels per second. The logical canvas is `BASE_W` × `BASE_H` (480 × 720), scaled by `resize()`. Coordinates should stay in that space.
- Don't block the frame loop. Avoid allocations in per-frame hot paths when a reuse is easy.
- Persist user settings only through the `STORAGE_*` keys, and wrap any new storage access so a blocked `localStorage` does not break the game.
- Keep the visible overlay and HUD text accessible. Announcements go through `announce()` into the `srLive` region.
- Respect `reducedMotion`: new screen shake, flashes, or large particle effects should check it. The setting defaults to the OS `prefers-reduced-motion` value until the player toggles it.

## Working in this repo

- The designated branch for this session is `claude/gifted-tesla-yipmn1`. Do not push to other branches without asking.
- Commit messages should describe the gameplay or code change in plain language, in the same style as the existing history (for example, "Add Sky Fighter: …").
