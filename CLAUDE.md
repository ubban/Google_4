# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 canvas application. No build system or external dependencies are required—the entire game runs in a modern web browser.

## How to Run

Open `index.html` in any modern web browser (desktop or mobile). The game works offline and uses only native browser APIs (Canvas, WebAudio, localStorage).

## Architecture

The game is monolithic (single HTML file) with three sections:

### 1. CSS (Lines 10–358)
- Responsive layout using viewport-relative units (`clamp()`, `env(safe-area-inset-*)`)
- Dark theme optimized for space/flight aesthetic
- Accessibility: reduced-motion media query, screen-reader live region, semantic buttons
- Mobile support: safe areas, touch event zones (joystick, fire, bomb buttons)

### 2. DOM & Input Setup (Lines 363–862)
- **Game UI elements**: HUD (score, lives, weapons), boss health bar, combo popups, overlay (title screen, pause, game over)
- **Input**: keyboard (arrows/WASD, Space fire, Shift/X bomb, P pause), mouse steering to canvas point, touch (left half for movement joystick, right half for fire/bomb buttons)
- **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

### 3. Game Engine (Lines 863–1666)
Single-threaded requestAnimationFrame loop → `update(dt)` → `draw()`.

**Core data structures:**
- `player`: position, speed, weapon state, invulnerability, cooldown
- `enemies[]`: spawn from top, patrolled downward, tile-based collision
- `boss`: health pool, attack phases, pattern-based bullet fire
- `bullets[]`, `enemyBullets[]`, `particles[]`, `powerUps[]`
- Game state: `score`, `lives`, `stage`, `chain`, `multiplier` (combo system)

**Key subsystems:**
- **Difficulty**: `DIFFICULTIES` object defines spawn rates, enemy speed, shoot chance, player lives, bombs, boss HP scaling (easy/normal/hard)
- **Enemy types**: `ENEMY_TYPES` with `grunt`, `interceptor` (fast), `tank` (durable). Each stage unlocks harder enemies via `minStage` field.
- **Boss patterns**: `BOSSES[]` defines 6 boss designs. `BOSS_PATTERNS[]` (indexed by boss design) defines 3 bullet-fire patterns each (radial, aimed, sweeps, homing, crosses). Boss telegraphs before each pattern with visual & audio cue.
- **Weapons**: `vulcan` (spread bullets, level 1–3 changes spread count) and `laser` (pierce-enabled, level 1–3 adds lanes). Weapon-type switch at max level.
- **Audio synthesis**: `beep()` (oscillator + filter) and `noiseBurst()` (procedural white noise). No external audio files; all SFX generated in real-time via WebAudio.
- **Scoring**: base score × multiplier (1–10). Multiplier increases every 5 consecutive kills; chain breaks after 2.2s of no kills.
- **Screen shake & effects**: `screenShake()`, `flashHit()`, `flashBomb()`, reduced motion respected system-wide.

## Common Tasks

### Add a new enemy type
1. Add entry to `ENEMY_TYPES` object (define `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`)
2. Update `spawnEnemy()` filter: ensure `minStage` allows it only after the appropriate stage
3. Add corresponding explosion SFX key to `EXPLODE_SFX` if you want custom explosion sound

### Add a new boss or modify an existing one
1. Edit `BOSSES[]` array (name, hull colors, core color, `hpMul` for difficulty scaling)
2. Add/edit corresponding pattern set in `BOSS_PATTERNS[]` at the same index
3. Each pattern is a function `(b) => {...}` that calls one of: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`

### Adjust game difficulty/balance
- Edit `DIFFICULTIES` object: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- Edit `KILLS_PER_STAGE` (18 kills to trigger boss)
- Modify base HP calculation in `spawnBoss()`: `70 + stage * 35`

### Change visuals
- **Color themes**: `THEMES[]` array (sky gradient stops, cloud/sun colors, theme name)
- **Enemy shapes**: `drawPlane()` function (player and enemy aircraft)
- **Boss shapes**: `drawBossShip()` function
- **UI colors & sizes**: CSS variables and hardcoded colors in draw calls (e.g., bullet colors `#ffee58`, enemy bullet `#ff5252`)

### Add audio SFX
Add a new function to the `sfx` object (e.g., `sfx.newSound = () => { beep(...); noiseBurst(...); }`). Call from game logic (e.g., enemy spawn, stage clear). Use `panFor()` helper to position audio left/right based on screen X.

## Code Organization Notes

- **Game loop**: `loop()` calls `update()` and `draw()` every frame. Frame counter `frame` used for animation and pattern timing.
- **Collision**: `rectsOverlap()` tests AABB (axis-aligned bounding box) overlap for all entities
- **Particles**: temporary visual feedback (created by `spawnExplosion()`, lifetime-based removal)
- **Invulnerability**: `player.invuln` cooldown; blinking visual during invuln period (line 1546)
- **Canvas scaling**: `resize()` function maintains 480×720 base, scales to fit window with 1.6× max; touch and UI elements use viewport-safe coordinates
- **Accessibility**: announcements via `announce()` (screen-reader live region), button labels, mobile-friendly touch zones

## Testing Notes

- Open in a modern browser (Chrome, Firefox, Safari, Edge)
- Test across devices: desktop, tablet, mobile
- Verify touch controls work: swipe left half to move, tap buttons to fire/bomb
- Check reduced-motion toggle (🚫 button) disables animations
- All game state persists via localStorage (best score, mute state, difficulty, motion preference)
