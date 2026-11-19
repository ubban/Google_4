# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single self-contained HTML5 canvas app.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, inline CSS, and an inline JavaScript IIFE. No build step, no external dependencies. |
| `README.md` | Minimal placeholder. |
| `CLAUDE.md` | This file. |

There is no package manager, bundler, test suite, or linter configured. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout inside `index.html`

- **`<style>`** (~lines 10–360): all CSS. Uses `:root { color-scheme: dark; }`, a fixed full-viewport `#container`, and a canvas that scales to fit (`max-width/max-height: 100%`). HUD, overlay card, and touch controls are absolutely positioned on top of the canvas.
- **`<body>` markup** (~lines 362–420): HUD (`#hud`), boss bar, combo/stage banners, top buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls`), and the start/game-over overlay (`#overlay`).
- **`<script>`** (~line 422 onward): one IIFE containing everything else:
  - **Persistence keys** (`skyfighter.*` in `localStorage`): best score, mute, difficulty, reduced motion. All storage access should stay tolerant of missing or blocked storage.
  - **Tuning tables**: `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs), `BOSS_PATTERNS` (bullet patterns), `KILLS_PER_STAGE`.
  - **Audio**: Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`, `panFor`). Audio is created lazily on first user gesture.
  - **Input**: keyboard (`keys[]`, `keydown`/`keyup`), mouse (steer/fire on canvas), and touch (virtual joystick, FIRE, BOMB).
  - **Game state and loop**: `resetGame`, `update(dt)`, `updateBoss(dt)`, spawn functions, collision via `rectsOverlap`, scoring with a kill-chain multiplier (`addScore`, `breakChain`), weapons (`fireWeapon`, levels 1–3), bombs (`useBomb`), and power-ups (`applyPowerUp`).
  - **Rendering**: `drawPlane`, `drawBossShip`, explosions, HUD helpers (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`), and screen shake/flash.
  - **Pause and resize**: `togglePause` (P / Esc / pause button) and `resize` (fits the canvas to a fixed `BASE_W × BASE_H` = 480 × 720 logical resolution).

## Controls (for reference when testing)

- Desktop: mouse moves and fires; arrows/WASD move; Space fires; X or Shift bombs; P or Esc pauses.
- Touch: drag left zone to fly; FIRE and BOMB buttons.
- Accessibility: a reduced-motion toggle disables screen shake, flash, and pulse effects. Its initial value follows `prefers-reduced-motion` unless the user has set it.

## Conventions

- **Keep the game self-contained.** Everything stays in `index.html` unless the user asks for a split. Do not introduce frameworks, CDNs, or build tooling without a request.
- **Preserve the existing style**: vanilla JS, `const`/`let`, 2-space indentation, single IIFE scope, and named constant tables for tuning values. Prefer adding or adjusting values in the existing tables over hard-coding numbers in logic.
- **Logical resolution is 480 × 720.** Coordinates in update/draw code use that space; the canvas is scaled by CSS and `resize()`.
- **Guard new storage reads/writes** and new audio calls so they don't throw when unavailable or before a user gesture.
- **Keep the game playable at phone width.** Touch controls must not break desktop layout, and vice versa.

## Making changes

- Read the relevant section of `index.html` before editing; the file is large (~1,500+ lines), so search for the function or constant name rather than reading it whole.
- Verify changes in a real browser when possible (start, play, pause, bomb, game over, and a stage transition). Syntax-check the inline script if no browser is available.
- Keep commits focused. Commit messages should describe the gameplay or UI change, not the implementation details of the edit.

## Git

- Default development branch for automated work: `claude/gifted-tesla-k5qki1`. Do not push to other branches without explicit permission.
- Do not open pull requests unless the user explicitly asks.
