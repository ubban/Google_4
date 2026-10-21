# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-file HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up with weapons, smart bombs, stages, and boss fights. The whole game lives in `index.html` (CSS, markup, and JavaScript inline). `README.md` is a placeholder.

There is no build step, no package manager, no dependencies, and no test suite.

## Running the game

- Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`) and visit it.
- Desktop: mouse to steer, click to fire. Arrow keys or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Touch: drag in the bottom-left area (`#moveZone`) to fly, tap FIRE, tap BOMB. Touch controls show only when the device reports touch support.

## File layout

- `index.html`, in this order:
  1. `<head>`: meta tags, an inline SVG favicon, and a `<style>` block with all CSS.
  2. `<body>`: the HUD (`#hud`), boss bar, combo and stage banners, top buttons (mute, reduced motion, pause), touch controls, and the start/pause overlay (`#overlay`).
  3. One `<script>` containing a single IIFE with `'use strict'`.
- `README.md`: placeholder only.

## Architecture of the game script

The script is organized into commented sections, marked with `// ---------- Name ----------`. Keep that convention when adding code.

- **Elements and constants**: cached DOM references, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage, cycling), `ENEMY_TYPES`, `BOSSES`.
- **Persistence**: best score, mute, difficulty, and reduced-motion preference are stored in `localStorage` under `skyfighter.*` keys. Every access is a direct call, so guard new storage reads and writes the same way if you add any.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`) and synthesized sound effects (`beep`, `noiseBurst`, and the `sfx` object). No audio files.
- **Canvas sizing**: the internal resolution is fixed at 480×720 (`BASE_W`, `BASE_H`) and scaled by CSS to fit the window, capped at 1.6×. Write all game coordinates in the 480×720 space.
- **Input**: keyboard state in `keys`, a virtual joystick for touch, and mouse/pointer handling through `canvasPointFromEvent`.
- **Game state**: `resetGame()` initializes the state. Key variables include `player` (weapon, level), `bullets`, `enemies`, `powerUps`, `bossActive`/boss state, `lives`, `bombCount`, `chain`, `multiplier`, `stage`, and `stageKills`.
- **Simulation**: `update(dt)` handles spawning, movement, collisions, scoring, chain/multiplier timing, and stage progression. Stage progression runs on kill counts: after `KILLS_PER_STAGE` (18) kills a boss warning fires, and a boss spawns.
- **Weapons**: `fireWeapon()` has two weapons, Vulcan (spread shots) and Laser (piercing lanes), each with 3 power levels. Power-ups in `applyPowerUp` raise the weapon level, and past level 3 they swap weapons.
- **Bosses**: `BOSS_PATTERNS` holds the attack patterns (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` drives them and `killBoss` advances the stage.
- **Rendering**: `draw()` renders the parallax sky, clouds, planes (`drawPlane`), bosses (`drawBossShip`), bullets, power-ups, and HUD effects.
- **Loop**: `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 0.05 s, skips updates while paused or not running, and calls `trackPerf`. `trackPerf` adjusts a `quality` level (0–2) from the average frame time over 45 frames.

## Difficulty tuning

Difficulty numbers live in the `DIFFICULTIES` table: spawn rate (`spawnBase`, `spawnMin`), enemy speed, enemy shoot chance, starting `lives` and `bombs`, and `bossHpMul`. Change balance there rather than in the logic. Enemy stats are in `ENEMY_TYPES` and boss stats in `BOSSES`.

Power-up drop chances are in `maybeSpawnPowerUp`: 6% extra life, 8% bomb, 20% weapon.

## Conventions

- Vanilla JavaScript only. Do not add frameworks, bundlers, or CDN dependencies unless the user asks for them.
- Keep everything inside the existing IIFE and the single `index.html` file. Do not split out modules or files without a request.
- Match the existing style: `const` and `let`, arrow functions for callbacks, two-space indentation, single quotes in JS, and short section comments.
- Respect reduced motion. Effects like screen shake, hit flash, and pulse are gated by the `reducedMotion` flag and the `reduced-motion` body class. New visual effects should honor it too.
- Keep the layout working on phones as well as desktops. Test both the mouse and touch paths when you change input or HUD layout.
- Accessibility: announce important state changes through `announce()`, which writes to the `#srLive` region.

## Verification

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Play through at least one stage, including a boss fight, on each difficulty you touched.
3. Check pause and resume, mute, reduced-motion toggle, and best-score persistence across a reload.
4. For layout or input changes, check a narrow mobile viewport with touch emulation.

If you add a test or lint setup, document the commands here.
