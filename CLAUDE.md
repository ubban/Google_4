# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas. The whole game is one self-contained file, `index.html`, with inline CSS and one inline `<script>`.

- No build step, no package manager, no dependencies, no tests.
- To run it, open `index.html` in a browser. Nothing needs to be installed.
- `README.md` is a placeholder and has no setup instructions.

## Layout of `index.html`

| Region | Approx. lines | Contents |
| --- | --- | --- |
| `<head>` / `<style>` | 1–360 | Meta tags, title, all CSS (HUD, overlay card, touch controls, reduced-motion styles) |
| `<body>` markup | 361–421 | Canvas, HUD, overlay/menu, touch controls, boss bar, `#srLive` live region |
| `<script>` | 422–end | One `(function () { 'use strict'; ... })()` IIFE with the entire game |

Inside the IIFE the code is grouped under `// ---------- Section ----------` banners. Keep new code in the matching section, and keep the banner style when adding a section.

Key sections in the script, in order:

1. **DOM refs and constants**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, storage keys.
2. **Adaptive performance**: `trackPerf` lowers visual `quality` (0 full, 1 medium, 2 low) when frame time drifts.
3. **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` table). No audio files are loaded.
4. **HUD and feedback**: hearts, bombs, combo pops, stage banner, screen shake, flashes.
5. **Layout and input**: `resize` (canvas is `BASE_W` × `BASE_H` = 480 × 720, scaled to fit), keyboard, touch joystick, fire and bomb buttons, mouse.
6. **Game state**: `resetGame`, `update(dt)`, spawning, collisions, scoring, bombs, weapons, power-ups.
7. **Bosses**: `BOSS_PATTERNS` are bullet-pattern functions; `updateBoss` and `killBoss` drive each fight.
8. **Flow**: `loseLife`, `endGame`, `togglePause`.
9. **Rendering**: `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
10. **Main loop**: `loop(now)` is a `requestAnimationFrame` loop that calls `update` and then `draw`.

## Game rules and where they live

- **Difficulty** (`DIFFICULTIES`): spawn rate (`spawnBase`, `spawnMin`), `enemySpeed`, `enemyShootChance`, starting `lives`, `bombs`, and `bossHpMul`. Easy, Normal, and Hard are selected from the menu buttons (`.diffBtn`).
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, which triggers `spawnBoss()` about 1.2 s later. Clearing a boss increments `stage`. `THEMES` cycles the sky palette per stage.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaves), `tank` (4 HP, appears from stage 2).
- **Bosses** (`BOSSES`): six named bosses with colours and `hpMul`. Each fight uses patterns from `BOSS_PATTERNS`.
- **Weapons**: `vulcan` (spread) and `laser` (piercing), each with 3 levels. See `fireWeapon()`.
- **Power-ups** (`applyPowerUp`): `life`, `bomb`, and `weapon`. The weapon power-up raises the level to 3, then switches weapon type and resets to level 1.
- **Score**: `addScore` multiplies by a kill-chain multiplier. `breakChain` resets it.

## Controls

| Action | Keyboard | Mouse | Touch |
| --- | --- | --- | --- |
| Move | Arrow keys or WASD | Ship follows the pointer over the canvas | Left-side joystick |
| Fire | Hold Space | Hold the mouse button | Fire button (hold) |
| Bomb | X or Shift | n/a | Bomb button |
| Pause | P or Esc | n/a | Pause button |

Touch controls are shown only when `'ontouchstart' in window` or `navigator.maxTouchPoints > 0`.

Space's default scroll behaviour is suppressed in the `keydown` handler. Keep that when changing input code.

## Persistence

Settings and best score are stored in `localStorage` under these keys. Changing a key name or its value format will silently reset players' saved data, so treat them as a public contract.

- `skyfighter.bestScore`: number
- `skyfighter.muted`: `'1'` or `'0'`
- `skyfighter.difficulty`: `'easy'`, `'normal'`, or `'hard'`
- `skyfighter.reducedMotion`: `'1'` or `'0'`. If unset, it follows `prefers-reduced-motion`.

Every `localStorage` access must keep working when storage is unavailable or throws. If you add a new read or write, handle that case.

## Accessibility and mobile

- Screen-reader announcements go through `announce()` into `#srLive`. Use it for important state changes such as pause, game over, and stage start.
- Reduced motion is a body class (`reduced-motion`) that disables shake, flashes, and similar effects. New visual effects should respect it.
- The layout must work at phone width. Touch controls are shown on touch devices. Keep the 16px gutter and avoid horizontal scrolling.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not add a build step or dependencies without asking.
- Match the existing style: 2-space indent, single quotes in JS, `const` by default, arrow functions for callbacks, and short comments only where the code is not obvious.
- Use the existing constants and tables (`ENEMY_TYPES`, `BOSSES`, `DIFFICULTIES`) for tuning values rather than hard-coding numbers in logic.
- Do not reference external CDNs or fetch remote assets. The game is fully offline.
- Keep `'use strict'` and the IIFE wrapper. Globals should not leak.

## Verifying changes

There is no automated test suite. Verify by hand:

1. Open `index.html` in a desktop browser and play through at least one full stage, including a boss fight, at each difficulty you touched.
2. Check the menu, pause (P and Esc), game over, and best-score persistence after a reload.
3. For HUD or layout changes, check a phone-sized viewport (for example, DevTools device mode) and confirm the touch controls still work.
4. For audio or input changes, confirm mute persists across a reload and that touch and keyboard both still fire and bomb.
5. Check the browser console for errors. A clean run is the minimum bar.

Headless browser checks are possible with a Playwright install, if one is available in the environment. Load `file:///.../index.html` and drive it with `keyboard.press`.

## Git

- Default branch: `main`.
- Commit messages are short imperative summaries, with an optional body explaining the why.
- Do not force-push shared branches.
