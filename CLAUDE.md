# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sky Fighter is a browser-based Raiden-style vertical shoot-'em-up game. The entire game is implemented as a single HTML file (`index.html`) with embedded CSS and JavaScript—no build process, dependencies, or external assets required.

## Running the Game

Simply open `index.html` in a modern web browser. The game runs directly without any build steps or server setup.

**Desktop controls:**
- **Move:** Mouse hover/drag, or Arrow Keys/WASD
- **Fire:** Mouse click or Space
- **Bomb:** X, Shift, or dedicated bomb button
- **Pause:** P or Esc

**Mobile:** Touch-based joystick on the left, FIRE and BOMB buttons on the right.

## High-Level Architecture

The game operates on a classic game loop pattern (line 1640):

1. **Input Phase** – Keyboard, mouse, and touch input update state variables (`keys`, `firing`, `joyVec`, `mouseTarget`)
2. **Update Phase** – Physics, collision detection, spawning, and state transitions
3. **Render Phase** – 2D Canvas drawing

All game state is stored in module-scope variables. The canvas is sized to fit the viewport while maintaining a 480×720 aspect ratio.

### Core Game State (lines 863–869)

- **Player:** Position, speed, weapon type/level, cooldown, invulnerability timer
- **Entities:** Arrays of bullets, enemies, enemy bullets, particles, power-ups, and boss
- **Game flow:** Score, lives, stage, multiplier chain, running/paused/gameOver flags

### Game Loop (`loop` function, lines 1640–1653)

Runs at 60 FPS via `requestAnimationFrame`. Delta time is clamped to 50ms to handle frame drops gracefully. When running and not paused, it calls `update(dt)` then `draw()`.

### Update Phase (`update` function, lines 1148–1302)

1. **Player movement** – Reads input and moves player with boundary clamping
2. **Weapons** – Fires bullets based on cooldown and selected weapon
3. **Enemy spawning** – Spawns enemies with difficulty scaling until stage boss threshold is met
4. **Entity updates** – Move bullets, enemies, particles; apply gravity to power-ups
5. **Collision detection** – Enemy-bullet, bullet-player, bullet-boss, power-up pickup
6. **Difficulty progression** – Advances stage when kill count reaches `KILLS_PER_STAGE` (18)

### Render Phase (`draw` function, lines 1520–1638)

- **Sky gradient** – Three-stop gradient per theme
- **Clouds** – Parallax scrolling layers using ellipses
- **Entities** – Player ship, enemies, bullets, particles, power-ups, boss
- **Effects** – Glow on high-quality setting, screen shake offset
- **HUD** – Rendered via DOM elements (score, weapon, health, etc.)

## Key Data Structures and Constants

### Difficulty Settings (lines 485–489)
Three presets: Easy, Normal, Hard. Control spawn rate, enemy speed, shoot chance, lives, bombs, and boss HP multiplier.

### Enemy Types (lines 502–506)
- **Grunt:** Fast, basic enemy (1 HP, 10 points)
- **Interceptor:** Fast weaver, medium threat (1 HP, 15 points)
- **Tank:** Slow, tanky (4 HP, 35 points, appears from stage 2)

Each has a movement pattern: grunts and tanks move straight; interceptors weave. Tanks can apply damage at scale.

### Boss Fleet (lines 508–515)
Six unique boss designs with distinct attack patterns. Each appears once per 6-stage cycle and scales by difficulty and stage.

### Boss Attack Patterns (lines 1372–1409)
Each boss has three attack patterns (rotation-based). Patterns include:
- **Radial bursts:** Bullets fire in all directions
- **Aimed spreads:** Bullets converge on player with spread
- **Homing volleys:** Bullets chase player position with noise
- **Columns, spirals, crosses:** Geometric patterns

### Themes (lines 493–500)
Six visual themes with sky gradients, cloud colors, and sun colors. Theme cycles by stage.

### Weapons (lines 871–1146)
- **Vulcan:** Spread shot, fast fire rate (0.15s cooldown), gains width at levels 2–3
- **Laser:** Single or multi-lane beam, slower fire rate (0.1s cooldown), pierces enemies, levels affect lane count

Power-ups randomly spawn and cycle weapon type or level.

### Audio System (lines 542–676)
Uses WebAudio API to synthesize all sound effects on the fly—no external audio files. Each SFX function creates oscillators, filters, and noise buffers. The `panFor()` function spatializes audio based on entity x-position.

**Key functions:**
- `beep(freq, dur, type, vol, opts)` – Tone with optional frequency sweep, filter, pan
- `noiseBurst(dur, vol, opts)` – White noise with filter

## Code Organization

The game is organized into logical sections within the IIFE (Immediately Invoked Function Expression):

1. **DOM elements & constants** (lines 426–489) – Cache references, define game constants
2. **Audio system** (lines 542–676) – Sound synthesis and playback
3. **Rendering helpers** (lines 680–730) – HUD updates, flash effects, combo popups
4. **Input handling** (lines 767–861) – Keyboard, mouse, touch, joystick logic
5. **Game state & reset** (lines 863–911) – Initialize and reset game
6. **Drawing functions** (lines 913–1020) – Render player, boss, clouds
7. **Spawning & physics** (lines 1022–1085) – Enemy/boss/power-up spawning, explosions
8. **Game mechanics** (lines 1087–1322) – Scoring, chain multipliers, power-ups, weapon fire
9. **Collision & update** (lines 1148–1302) – Main update loop and collision detection
10. **Boss AI** (lines 1324–1434) – Boss-specific attack patterns and behavior
11. **Game flow** (lines 1441–1505) – Win/lose/pause logic
12. **Main loop** (lines 1640–1666) – RAF loop and startup

## Extending the Game

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` (line 502) with width, height, HP, color, and score
2. Update `spawnEnemy()` (line 1022) to weight the new type into the random selection
3. Add explosion SFX mapping in `EXPLODE_SFX` (line 678) if desired
4. Update collision and rendering code if special behavior is needed

### Adding a New Weapon

1. Define fire pattern in `fireWeapon()` (line 1124)
2. Update `weaponLabel()` (line 871) to return the display name
3. Update power-up cycling logic in `applyPowerUp()` (line 1304) if weapon should cycle with level

### Adding a Boss Pattern

1. Create a new pattern function (e.g., `fireSpecialAttack(b)`) using helper functions like `fireRadial()`, `fireAimedSpread()`, etc.
2. Add the pattern array to `BOSS_PATTERNS` (line 1372)
3. Optionally add a new boss design to `BOSSES` (line 508) to use the pattern

### Performance Tuning

The game adapts rendering quality based on frame time:
- **Quality 0 (full):** All particles, full glow effects, max shadow blur
- **Quality 1 (medium):** 60% particle density, reduced glow
- **Quality 2 (low):** 35% particle density, no glow effects

Adjust thresholds in `trackPerf()` (line 476) if needed. On very fast devices, increase from 40fps/56fps targets. On slow devices, lower the thresholds.

### Adding Visual Effects

- **Screen shake:** Call `screenShake(magnitude, duration)` from anywhere in update
- **Explosions:** Call `spawnExplosion(x, y, color, particleCount)` to emit particles
- **Combo popups:** Call `showComboPop(text)` or `showStageBanner(text)` for overlay messages
- **Flash effects:** Call `flashHit()` or `flashBomb()` to trigger screen flash animation

## Testing & Debugging

Since there's no automated test framework, manual testing is the standard approach:

1. **Feature testing:** Play through multiple stages on each difficulty to verify new content
2. **Browser console:** Use `console.log()` within the game code; check DevTools for errors
3. **Accessibility:** Test reduced-motion preference, screen reader announcements via `announce()`
4. **Mobile:** Test touch controls on actual devices or use DevTools device emulation

## Notes on Browser Compatibility

- Requires modern Canvas 2D, WebAudio API, and `requestAnimationFrame`
- Works on Chrome, Firefox, Safari, Edge (desktop and mobile)
- Respects `prefers-reduced-motion` media query for accessibility
