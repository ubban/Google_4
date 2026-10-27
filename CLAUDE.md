# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 canvas. The entire game is one file, `index.html`, which contains the markup, inline CSS, and one inline `<script>` block. There is no build step, no package manager, no bundler, no external assets, and no test suite. `README.md` is a placeholder and does not describe the project.

## Layout

| Lines (approx.) | Contents |
| --- | --- |
| 1–9 | `<head>`: meta tags (viewport locked, `theme-color`), `<title>` |
| 10–360 | Inline `<style>`: HUD, overlay, touch controls, responsive layout |
| 361–421 | `<body>`: canvas, HUD elements, overlay (start/pause/game over), touch controls |
| 422–1667 | Single `<script>` IIFE (`'use strict'`) containing all game logic |

Inside the script, code is grouped by `// ---------- Section ----------` comments. The main sections, in order, are: DOM lookups and constants, adaptive performance, difficulty / theme / enemy / boss tables, persisted settings, audio (WebAudio synth), HUD rendering, resize, input (keyboard, touch joystick, fire/bomb buttons, mouse), game state and entities, `update(dt)`, boss patterns, `draw()`, and the main `loop(now)`.

## Key constants and tables

- **Canvas:** logical size `BASE_W = 480`, `BASE_H = 720`, scaled to fit the viewport by `resize()`.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy shoot chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages:** `KILLS_PER_STAGE = 18` controls stage pacing. `THEMES` holds six sky palettes and names.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
- **Bosses** (`BOSSES`): six bosses with their own colors and HP multipliers, attacked with patterns from `BOSS_PATTERNS`.
- **Weapons:** `vulcan` and `laser`, each with up to 3 levels. Collecting a weapon power-up raises the level; past level 3 it switches weapon and resets to level 1 (`applyPowerUp`).
- **Power-ups:** `life`, `bomb`, `weapon`.

## Persistence

Settings and high score are stored in `localStorage` under these keys. Keep the `skyfighter.` prefix for any new key.

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Reads and writes are not wrapped in try/catch. If you add storage access that might run in a restricted context (private mode, blocked site data), guard it.

## Controls

- **Keyboard:** arrow keys or `WASD` move (read in `update()`); `Space` fires; `X` / `Shift` drop a bomb; `P` / `Escape` toggle pause.
- **Mouse:** move the pointer over the canvas to steer; mouse-down on the canvas or the on-screen FIRE button fires (the on-screen button uses `mousedown`/`mouseup` on `window`).
- **Touch:** left-side joystick zone for movement; FIRE and BOMB buttons on the right. Touch handlers call `preventDefault` to avoid scrolling.

## Runtime behavior worth knowing

- **Game loop:** `loop(now)` is a `requestAnimationFrame` loop. It clamps `dt` to 0.05 s and only calls `update` and `draw` while the game is running and not paused.
- **Adaptive quality:** `trackPerf` averages the last 45 frame times and steps a `quality` level (0 full, 1 medium, 2 low) down when frames are slow and back up when they are fast.
- **Reduced motion:** the user can toggle it with the 🎆 button. It defaults from `prefers-reduced-motion` and gates screen shake, flashes, and pulses.
- **Audio:** all sound is synthesized with WebAudio (`beep`, `noiseBurst`, the `sfx` table). `ensureAudio()` creates the context lazily on the first user gesture, so keep that call on any new input path. Mute is persisted.
- **Accessibility:** `announce(text)` writes to the `srLive` live region. Use it for important state changes such as stage start, game over, and new best.

## Conventions

- Match the existing style: 2-space indentation, single quotes, semicolons, `const`/`let`, no classes, and everything inside the one IIFE. Do not introduce globals.
- Keep it a single file unless the user asks for a split. Do not add a build system, dependencies, or a framework without an explicit request.
- Name new tables and constants in `UPPER_SNAKE_CASE`, and place them with the related table near the top of the script.
- Add new enemy, boss, or power-up data as entries in the existing tables rather than adding special cases in `update()` or `draw()`.
- Keep comments short and only where the logic is not obvious.

## Working on the game

There is no lint, test, or build command. To verify a change:

1. Open `index.html` directly in a browser, or serve the directory with any static server.
2. Play through the affected path: start, fire, bomb, pause, a power-up pickup, and game over. For bosses or stage transitions, adjust `KILLS_PER_STAGE` or the spawn logic temporarily if needed, and revert it afterward.
3. Check the mobile layout with touch emulation, since the touch controls and viewport lock are part of the design.
4. Check the console for errors. Headless Chromium with Playwright is available in this environment and can drive the page for automated checks.

Because the file is large (about 1,670 lines), prefer targeted edits. Use the section comments above to find the right area.
