# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single self-contained HTML5 canvas page.

- `index.html` is the entire game: markup, inline CSS, and an inline `<script>` (no build step, no bundler, no external dependencies).
- `README.md` is a placeholder with only the project title.

There is no `package.json`, test suite, linter, or CI configuration. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

```
index.html   Game: HTML (canvas, HUD, overlay, touch controls), CSS (<style>), JS (<script>)
README.md    Project title
CLAUDE.md    This file
```

Inside `index.html`, the script is one strict-mode IIFE that holds all game state and logic. The main sections, roughly in file order:

| Area | What it contains |
|---|---|
| Config constants | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES` |
| Persistence | `localStorage` keys prefixed `skyfighter.` (best score, mute, difficulty, reduced motion) |
| Audio | Lazily created `AudioContext`; `beep`, `noiseBurst`, and the `sfx` table. Audio starts on first user input. |
| UI feedback | Hearts/bombs rendering, hit/bomb flashes, combo popups, stage banner, screen shake, live-region announcements |
| Input | Keyboard, touch joystick plus fire/bomb buttons, and mouse on the canvas |
| Game state | `resetGame()`; `update(dt)`, `updateBoss(dt)`, `draw()`, and `loop(now)` drive the frame |
| Rendering helpers | `drawPlane`, `drawBossShip`, `drawClouds` |

## Game rules and tuning

Values to check first when changing balance:

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, then a boss spawns. Bosses cycle through `BOSSES` by stage index, and boss HP is `70 + stage * 35` times the boss and difficulty multipliers.
- **Enemies**: `grunt`, `interceptor` (fast, weaving), and `tank` (high HP, unlocked at stage 2). `minStage` gates each type.
- **Power-ups**: `life`, `bomb`, and `weapon` (raises weapon level, up to three levels of Vulcan/Laser).
- **Scoring**: kill-chain combo multiplier (`chain`, `chainTimer`, `multiplier`), and best score persists.

## Controls

| Action | Keyboard | Touch |
|---|---|---|
| Move | WASD / arrow keys | Left-side joystick |
| Fire | Space (held) | Fire button (held) |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Esc | Pause button |
| Mute | Mute button | Mute button |

Mouse input moves the ship on the canvas, and holding the mouse button on the canvas fires. The game is scaled to fit the window, with a base logical size of `480 x 720` and a scale cap of 1.6.

## Conventions

- **Keep it one file.** Put new markup, styles, and script in `index.html` unless the user asks for a split. Don't add a build step, bundler, or frameworks without asking.
- **Match the existing style.** Use 2-space indentation, `const`/`let`, plain DOM APIs, and short comments only where they explain something non-obvious. Name new config entries in the same shape as the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`).
- **Respect accessibility.** Reduced motion is honored via `prefers-reduced-motion` and a user toggle. Any new screen shake, flash, or animation should go through the existing `reducedMotion` checks. Keep the `srLive` announcements in sync with important state changes.
- **Mobile first.** The layout must work at phone width and with touch input. Test touch changes with touch emulation, not just the mouse.
- **Persistence is local only.** Use the `skyfighter.*` `localStorage` keys and wrap reads and writes in a way that tolerates storage being unavailable.

## Verification

No automated checks exist. For any change, verify by hand:

1. Open `index.html` in a desktop browser and play through at least one full stage, including the boss.
2. Check each difficulty you touched, and confirm the best score still saves across a reload.
3. Check touch controls in a mobile-sized viewport when input or layout changed.
4. Toggle mute, motion, and pause, and confirm the HUD still updates.
5. Watch the browser console for errors. A clean console is part of "done".

If you add tooling later (tests, a linter, a build), update this section with the exact commands.

## Git

- Develop on the branch specified by the session, and push with `git push -u origin <branch>`.
- Write commit messages in the imperative mood with a short subject line and a body that explains the change.
