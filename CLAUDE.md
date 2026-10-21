# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up played in the browser on an HTML5 `<canvas>`.

Files:

- `index.html` (~1,670 lines) — the entire game: markup, CSS, and JavaScript in one self-contained file.
- `README.md` — a one-line title. Keep it short; put real docs here only if asked.

There is no build step, no package manager, no test suite, and no linter configured.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

No install is needed. Verify changes by playing them in a browser; there are no automated checks to run.

## Code layout of `index.html`

- **`<style>`** (around lines 10–360): all CSS, including the HUD pills, the glassmorphic overlay card, the touch controls, and the `.reduced-motion` class.
- **`<body>`** (around lines 361–421): the DOM skeleton. Elements are looked up by `id` in the script, so renaming an `id` breaks the game silently. Key ids: `gameCanvas`, `overlay`, `startBtn`, `score`, `hearts`, `bombs`, `bossBarWrap`, `touchControls`, `fireBtn`, `bombBtn`.
- **`<script>`** (around lines 422–1668): one IIFE with `'use strict'`. Nothing is exported to `window`.

Inside the script, roughly in order:

1. DOM references and `STORAGE_*` localStorage keys.
2. Tuning constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
3. Audio: `ensureAudio`, `beep`, `noiseBurst` (Web Audio, created lazily on the first user gesture).
4. HUD and effect helpers: `renderHearts`, `renderBombs`, `flashHit`, `screenShake`, `showComboPop`, `showStageBanner`.
5. Input and layout: `resize` (fixed `480×720` logical size, scaled to fit), joystick and pointer handling.
6. Game state: `resetGame`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`), bombs (`useBomb`), scoring (`addScore`, `breakChain`).
7. Simulation: `update(dt)` for the player, enemies, bullets, and collisions; `applyPowerUp`; boss logic (`BOSS_PATTERNS`, `updateBoss`, `killBoss`); life and game-over (`loseLife`, `endGame`).
8. Rendering: `draw()` with helpers `drawPlane`, `drawBossShip`, `drawClouds`.
9. Main loop: `loop(now)` — a `requestAnimationFrame` loop that clamps `dt` to 0.05 s, runs `trackPerf`, then `update` and `draw` unless paused.

### Conventions to preserve

- **Fixed-timestep-ish updates.** All movement and timers use `dt` in seconds. Do not use frame counts for gameplay timing.
- **Adaptive quality.** `trackPerf` steps a `quality` level (0–2) based on average frame time. New expensive effects should respect it.
- **Reduced motion.** Screen shake, hit flash, and pulses must check the `reducedMotion` flag (`screenShake` already clamps duration when it is set).
- **Persistence.** Player settings and best score live in `localStorage` under the `skyfighter.*` keys. Always go through the existing `STORAGE_*` constants. Wrap new storage access so the game still works if storage is blocked.
- **Accessibility.** Announce state changes through `announce()` into the `#srLive` live region.
- **Pause.** Gameplay freezes when `paused` or `!running`; pausing must not stop the rAF loop.

## Controls (for reference when testing)

- Desktop: mouse to steer and click to fire; arrows/WASD to move; Space to fire; X or Shift for a bomb; P or Esc to pause.
- Touch: drag in the left zone for the joystick; tap FIRE and BOMB.
- Difficulty (Easy / Normal / Hard) is chosen on the overlay and persists.

## Making changes

- Keep the project a single self-contained HTML file unless the user asks to split it. Do not add a bundler, framework, or package.json without a request.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`, arrow callbacks, and short section comments in the `// ---------- Name ----------` form.
- Prefer small, local edits. Game balance numbers (spawn rates, HP multipliers, scores) are intentionally kept in the constant tables near the top of the script; change them there rather than hard-coding values elsewhere.
- When you add a new enemy, boss, theme, or power-up, extend the existing table or pattern list and check every place that switches on its type (spawn, update, draw, and scoring).
- Test in a real browser after any gameplay or rendering change, at both desktop and phone-sized viewports, since the touch layout is separate.

## Git

- Development branch for automated work: `claude/gifted-tesla-erd1hw`. Default branch: `main`.
- Commit with clear, descriptive messages that explain why the change was made.
- Do not open pull requests unless asked.
