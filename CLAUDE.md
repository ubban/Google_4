# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML file with no build step, no package manager, and no dependencies.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (`<body>`), CSS (`<style>`, ~lines 10–362), and JavaScript (`<script>`, ~lines 422–1668). |
| `README.md` | Placeholder title only. |

There is no test suite, linter, or CI configuration. Verify changes by opening `index.html` in a browser.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- No install step is needed.

## Architecture of `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })();`). Nothing is exported to `window`, so keep new code inside that wrapper.

Major sections, in file order:

1. **DOM references**: `const xxxEl = document.getElementById(...)` at the top of the script. Add new element lookups here.
2. **Storage keys**: `STORAGE_*` constants prefixed `skyfighter.` (best score, mute, difficulty, reduced motion). Every `localStorage` access should be read through these keys.
3. **Accessibility helpers**: `announce()` writes to the `#srLive` live region for screen readers.
4. **Adaptive performance**: `trackPerf()` samples frame time and bumps a `quality` level (0 full, 1 medium, 2 low) when frames are slow or fast.
5. **Tuning data**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, and `BOSSES`. Balance changes usually belong here rather than in the update logic.
6. **Audio**: `ensureAudio()` lazily creates the `AudioContext` (browsers require a user gesture first). `beep()` and `noiseBurst()` synthesize every sound effect; there are no audio files. `sfx.*` is the public surface. Call `ensureAudio()` from input handlers before playing sound.
7. **HUD and feedback**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`.
8. **Settings UI**: mute, reduced-motion, and difficulty buttons. Each persists its value to `localStorage`.
9. **Layout**: `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport. Game coordinates always use the base size.
10. **Input**: keyboard (`keys[]` map, `keydown`/`keyup`), touch joystick plus Fire/Bomb buttons, and mouse steering/firing on the canvas. Keyboard and joystick take priority when active.
11. **Game state**: `resetGame()` initializes state. `update(dt)` advances the simulation; `draw()` renders it. Both run from `loop(now)` through `requestAnimationFrame`, and `dt` is clamped to 50 ms.
12. **Weapons, power-ups, and bombs**: `fireWeapon()`, `applyPowerUp()`, `useBomb()`.
13. **Bosses**: `spawnBoss()`, `updateBoss()`, `killBoss()`, with bullet-pattern helpers (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, etc.) collected in `BOSS_PATTERNS`.
14. **Lifecycle**: `loseLife()`, `endGame()` (writes the best score), `togglePause()`.

Conventions:
- Entities are plain objects in arrays; collision uses `rectsOverlap(a, b)`.
- Time-based motion multiplies by `dt`. Do not tie speed to frame count.
- Respect `reducedMotion` for new screen-shake, flash, or particle effects.
- Keep touch targets and the on-screen controls working when changing input code; `touchControls` is shown on touch devices.

## Conventions for editing

- Match the existing style: 2-space indent, single quotes, `const`/`let`, section headers written as `// ---------- Name ----------`.
- Keep everything in `index.html` unless the project is intentionally split. Do not add a bundler, framework, or package.json without asking.
- Don't add external CDN scripts or fonts without a reason; the game is designed to work offline from one file.
- When changing a tuning constant, check every place it is used (difficulty, boss HP multipliers, and spawn timing interact).

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-b8bv62`). Do not push to other branches without explicit permission.
- Commit with clear, descriptive messages and push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
