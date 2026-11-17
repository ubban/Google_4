# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, Raiden-style vertical shoot-em-up
built as an HTML5 canvas game. There is no build step, no package manager, and no
test suite.

- `index.html` — the entire game: inline CSS (`<style>`), markup (HUD, overlays,
  touch controls) and one inline `<script>` wrapped in an IIFE with `'use strict'`.
- `README.md` — placeholder; contains only the repo title.

To run the game, open `index.html` in a browser. No server is required.

## Repository layout and conventions

- Keep the game self-contained in `index.html`. Do not introduce bundlers, frameworks
  or external CDN dependencies unless the user asks for them.
- Keep the code inside the existing IIFE. Do not add globals.
- Use `const` / `let`, `function` declarations, and the existing naming style
  (camelCase for functions and variables, UPPER_SNAKE for constant tables such as
  `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, and `STORAGE_*` keys).
- Persistent state uses `localStorage` with keys prefixed `skyfighter.`
  (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`,
  `skyfighter.reducedMotion`). Wrap new storage keys in the same style.
- Rendering is done in `draw()` with the 2D canvas context; game state is updated in
  `update(dt)`. The main loop is `loop(now)`, which clamps `dt` to 0.05 s.

## Code map (index.html)

Approximate line numbers; they will drift as the file changes.

| Area | Where to look |
| --- | --- |
| DOM references and storage keys | top of the IIFE, ~L423–470 |
| Difficulty presets (`DIFFICULTIES`) | ~L485 |
| Stage themes (`THEMES`), enemies (`ENEMY_TYPES`), bosses (`BOSSES`) | ~L493–520 |
| Audio (`ensureAudio`, `beep`, `noiseBurst`) | ~L545–680 |
| HUD helpers (hearts, bombs, combo pops, stage banner, screen shake) | ~L680–735 |
| Responsive canvas sizing (`BASE_W = 480`, `BASE_H = 720`, `resize`) | ~L753 |
| Touch joystick and fire/bomb buttons | ~L785–840 |
| Game reset and entities (`resetGame`, `drawPlane`, `spawnEnemy`, `spawnBoss`) | ~L875–1070 |
| Weapons and bombs (`fireWeapon`, `useBomb`) | ~L1103–1150 |
| Main update (`update`), power-ups (`applyPowerUp`) | ~L1148–1325 |
| Boss attack patterns (`BOSS_PATTERNS`, `updateBoss`, `killBoss`) | ~L1372–1455 |
| Lives, game over, pause (`loseLife`, `endGame`, `togglePause`) | ~L1456–1505 |
| Rendering (`draw`, `drawClouds`) | ~L1507–1640 |
| Main loop and start button | ~L1640+ |

## Gameplay and controls (for reference when changing behavior)

- **Movement:** Arrow keys or WASD; mouse/touch drag on the canvas; on-screen joystick on touch devices.
- **Fire:** Hold Space, or hold the fire button (mouse or touch).
- **Bomb:** `X` or either Shift key, or the bomb button. Bombs clear enemy bullets and enemies and damage bosses.
- **Pause:** `P` or `Escape`, or the pause button.
- **Mute / reduced motion:** HUD buttons. Reduced motion disables screen shake, flashes and pulses.
- **Difficulty:** `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Progression:** Each stage needs `KILLS_PER_STAGE` (18) kills, then a boss fight. Six bosses and six visual themes cycle through the stages.
- **Scoring:** Kill-chain score multiplier that resets via `breakChain()`.
- **Weapons:** Vulcan / Laser, three power levels, raised by power-ups.
- **Adaptive quality:** `trackPerf` lowers the `quality` level when frame time is high (<40 FPS) and raises it when frame time is comfortably fast (>56 FPS). Keep new visual effects compatible with this scaling.

## Making changes

- Read the surrounding code before editing; this file is large (~1,650 lines) and most logic lives in one IIFE.
- Preserve the mobile/touch paths when touching input code. Both keyboard and touch are supported.
- Preserve accessibility hooks: `srLive` announcements via `announce()`, and the `reduced-motion` body class.
- Do not add new dependencies, build tooling, or test frameworks unless asked.

## Verification

There are no automated tests or linters in this repo. To verify a change:

1. Open `index.html` in a browser (or use the pre-installed Playwright/Chromium for a headless check).
2. Confirm the start overlay appears, the game starts on Start, and the HUD updates.
3. Exercise the code path you changed (for example, a boss fight, a power-up, pause/resume, or game over).
4. Check the browser console for errors.

If you cannot run the game, say so explicitly and state what was verified by reading the code.

## Git

- Development branch for automated work: `claude/gifted-tesla-qj4b4z`.
- Commit messages should be short and imperative, with a body when the change needs context (the existing commit log follows this style).
